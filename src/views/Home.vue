<template>
    <div class="home">
        Scalar param: {{ scalarParam }} <br /> <br />
        Scalar param (validation $model): <input type="text" v-model="$v.scalarParam.$model" /> <br /> <br />

        Object prop (should not be changed by the reference): {{ objectProp }} <br /> <br />
        Object param (initial value from the prop, only local changes): {{ objectParam }} <br /> <br />

        Computed param: {{ computedParam }}
    </div>
</template>

<script>
import { required } from 'vuelidate/lib/validators';

export default {
    props: {
        scalarProp: {
            type: Number,
            default: 0,
        },
        objectProp: {
            type: Object,
            default: () => (
                {
                    b: [
                        {
                            a: [1, 2, 3],
                            b: [4, 5, 6],
                            c: [7, 8, 9],
                        },
                    ],
                }
            ),
        },
    },
    data() {
        // We can use props here, but not computed yet.
        console.log('1. Data parameters evaluation.');

        // We can't use a computed parameter here.
        console.log('Trying to use a computed param: ', this.computedParam, ' ...nope!');

        // We should also perform a deep clone on objects and arrays here.
        // Object.assign: not a deep clone (depth 1):
        // let B1 = Object.assign({}, this.objectProp);
        //
        // delete B1.b[0].b;
        // console.log('Bad clone: ', this.objectProp); // this.objectProp.b[0].b is also gone.

        // JSON.stringify + JSON.parse: a deep clone
        const B1 = JSON.parse(JSON.stringify(this.objectProp));

        delete B1.b[0].b;
        console.log('Good clone (deep, the source prop is not modified): ', this.objectProp);

        return {
            scalarParam: this.scalarProp,
            objectParam: B1,
        };
    },
    validations: {
        scalarParam: {
            required,
        },
    },
    computed: {
        computedParam() {
            console.log('3. Computed param evaluation.');

            return 'computedParam';
        },
    },
    watch: {
        scalarParam: {
            handler(valueNew, valueOld) {
                // This will evaluate a computed param.
                // console.log(this.computedParam);

                console.log('2. Watch on data parameter (with immediate: true).');
            },
            immediate: true,
        },
        // scalarParam: {
        //     handler(valueNew, valueOld) {
        //         console.log('6. Watch on data parameter (with immediate: false).');
        //     },
        // },
        computedParam: {
            handler(valueNew, valueOld) {
                console.log('4. Watch on computed parameter (with immediate: true).');
            },
            immediate: true,
        },
    },
    // todo: other lifecycle hooks.
    mounted() {
        console.log('5. mounted() evaluation.');

        // It is possible to use a computed param here.
        // console.log('Trying to use a computed param: ', this.computedParam);

        // Will be an initial value from the data() - 0.
        // console.log(this.$v.scalarParam.$model);

        const self = this;

        // To emulate a data change action during the lifecycle.
        setTimeout(
            function () {
                self.scalarParam = 1;
            },
            100,
        );
    },
    beforeUpdate() {
        // 1
        console.log(this.$v.scalarParam.$model);
    }
};
</script>
