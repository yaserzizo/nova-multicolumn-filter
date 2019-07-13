<template>
    <div>
        <h3 class="text-sm uppercase tracking-wide text-80 bg-30 p-3">{{ filter.name }}</h3>

        <div class="">
            <div v-for="(value, index) in rows" class="flex p-2">
                <Column :columns="columns"
                        :operators="operators"
                        :column="value.column"
                        :operator="value.operator"
                        :search="value.search"
                        :index="index"
                        @change="childChange"
                >
                </Column>
                <button @click="removeColumn(index)" class="btn btn-block p-1 rounded ml-1 btn-danger">X</button>
            </div>

            <div class="p-2">
                <button @click="addColumn" class="btn btn-default btn-primary">Add</button>
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
            operators : [],
        }
    },

    mounted() {
        let t = JSON.parse(this.options[0].value);
        for (let i in t.columns) {
            if (t.columns[i] instanceof Object) {
                this.columns.push({
                    label: t.columns[i].label,
                    value: i,
                    type: t.columns[i].type,
                });
            } else {
                this.columns.push({
                    label: t.columns[i],
                    value: i,
                    type: 'text',
                });
            }
        }

        for (let i in t.operators) {
            this.operators.push({
                label: t.operators[i],
                value: i,
            });
        }

        try {
            this.rows = JSON.parse(this.value);
        } catch (e) {
            this.rows = [];
        }
    },

    methods: {
        handleChange() {
            for (let i = 0; i < this.rows.length; i++) {
                if (!this.rows[i].column || !this.rows[i].operator || !this.rows[i].search) {
                    return;
                }
            }

            let value = this.rows.length === 0 ? '' : JSON.stringify(this.rows);

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
                search: '',
            });
        },

        removeColumn(index) {
            this.rows.splice(index, 1);
            this.handleChange();
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