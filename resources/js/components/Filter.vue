<template>
    <div>
        <h3 class="text-sm uppercase tracking-wide text-80 bg-30 p-3">{{ filter.name }}</h3>

        <div class="">
            <div v-for="(value, index) in rows" :key="value.value" class="flex p-2 w-full">
                <Column :columns="columns"
                        :operators="operators"
                        :column="value.column"
                        :operator="value.operator"
                        :value="value.value"
                        :index="index"
                        @change="childChange"
                >
                </Column>
                <div @click="removeColumn(index)" class="btn btn-block p-1 rounded ml-1 btn-danger close-button cursor-pointer">X</div>
            </div>

            <div class="p-2">
                <div @click="addColumn" class="btn btn-default btn-primary cursor-pointer">Add</div>
                <div
                    v-if="manualUpdate"
                    @click="update"
                    class="btn btn-default btn-primary cursor-pointer"
                >
                    Apply
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Column from './Column';

export default {
    props: {
        resourceName: {
            type: String,
            required: true,
        },
        filterKey: {
            type: String,
            required: true,
        },
    },

    components: {
        Column,
    },

    data() {
        return {
            rows: [],
            columns: [],
            operators: [],
            manualUpdate: false,
        }
    },

    mounted() {
        let t = JSON.parse(this.options[0].value);

        this.columns = t.columns;
        this.manualUpdate = t.manual;

        try {
            this.rows = JSON.parse(this.value);
        } catch (e) {
            this.rows = [];
        }
        if (this.rows.length === 0) {
            for (let i in t.columns) {
                if (t.columns[i].preset) {
                    this.rows.push({
                        column: i,
                        operator: t.columns[i].defaultOperator || '',
                        value: encodeURIComponent(t.columns[i].defaultValue || ''),
                    });
                }
            }

            this.handleChange();
        }

    },

    methods: {
        handleChange(manual = false) {
            if (this.manualUpdate && !manual) {
                return;
            }

            for (let i = 0; i < this.rows.length; i++) {
                if (!this.rows[i].column || !this.rows[i].operator || this.rows[i].value === '') {
                    return;
                }
            }

            let value = this.rows.length === 0 ? '' : JSON.stringify(this.rows);
          //  this.rows = value;
            this.$store.commit(`${this.resourceName}/updateFilterState`, {
                filterClass: this.filterKey,
                value: value,
            });

            this.$emit('change');
        },

        childChange(index, data) {
            this.rows[index] = data;
            this.handleChange();
        },

        addColumn() {
            this.rows.push({
                column: '',
                operator: '',
                value: '',
            });
        },

        removeColumn(index) {
            this.rows.splice(index, 1);
            this.handleChange();
        },

        update() {
            this.handleChange(true);
        },
    },

    computed: {
        filter() {
            return this.$store.getters[`${this.resourceName}/getFilter`](this.filterKey)
        },

        value() {
            return this.filter.currentValue
        },

        options() {
            return this.$store.getters[`${this.resourceName}/getOptionsForFilter`](this.filterKey)
        },
    },
}
</script>

<style scoped>
    .close-button {
        display: flex;
        justify-content: center;
        align-items: center;
    }
</style>
