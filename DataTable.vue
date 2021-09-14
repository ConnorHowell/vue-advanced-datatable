<template>
    <div class="vue-datatable">
        <div class="d-flex justify-content-between align-items-center push">
            <div class="flex-grow-1 d-flex align-items-center">
                <div class="input-group" style="width: 24rem;">
                    <div class="input-group-prepend">
                        <button type="button" class="btn btn-secondary" @click="searchTerm = null">
                            <i class="fa fa-times"></i>
                        </button>
                    </div>
                    <input v-model.lazy="searchTerm" type="text" class="form-control" placeholder="Search data...">
                    <div class="input-group-append">
                        <button type="submit" class="btn btn-secondary">
                            <i class="fa fa-search"></i>
                        </button>
                    </div>
                </div>
            </div>

            <div class="d-flex align-items-center">
                <div class="spinner-border text-secondary mr-10" role="status" v-show="busy">
                    <span class="sr-only">Loading...</span>
                </div>

                <div class="mr-5">
                    <button class="btn btn-outline-success" data-toggle="click-ripple" @click="exportData()"><i class="far fa-file-excel mr-5"></i> Export</button>
                </div>

                <div class="mr-5">
                    <button class="btn btn-info" data-toggle="click-ripple" v-b-modal="`modal-datatable-filter-${_uid}`"><i class="fas fa-filter mr-5"></i> Filter Data</button>
                </div>

                <div class="mr-5">
                    <button class="btn btn-secondary" data-toggle="click-ripple" @click="toggleTableView()"><i class="fas fa-desktop mr-5"></i> Toggle View</button>
                </div>

                <div class="flex flex-col items-center">
                    <div class="dropdown">
                        <button class="btn btn-secondary dropdown-toggle" type="button" @click="showColumnDropdown = !showColumnDropdown">
                            Show / Hide Columns <span class="font-w700" v-if="hiddenFields.length > 0">({{ immutableColumns.length - hiddenFields.length }}/{{ immutableColumns.length }})</span>
                        </button>
                        <div :class="{'dropdown-menu': true, show: showColumnDropdown}">
                            <button class="dropdown-item" type="button" @click="toggleColumn(col)" v-for="col in immutableColumns" :key="col">
                                <i :class="`far ${ (hiddenFields.includes(col)) ? 'fa-times-circle text-danger' : 'fa-check-circle text-success' } mr-5`"></i>
                                {{ col | heading }}
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div :class="{'table-responsive': responsive, 'border': bordered, 'wrapper': stickyHeader}">
            <table :class="tableClass" id="report-output">
                <thead :class="theadClass">
                    <tr>
                        <th :data-header="col" v-for="col in columns" :key="col" @click="toggleSort(col)" class="shadow-sm">
                            <span class="d-flex align-items-center  justify-content-between">
                                {{ col | heading }}
                                <i :class="{'fas': true, 'fa-sort': (!isSorted(col)), 'fa-sort-up': (isSorted(col) && !sort.desc), 'fa-sort-down': (isSorted(col) && sort.desc)}"></i>
                            </span>
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-show="!results.length">
                        <td :colspan="columns.length" class="text-center text-muted">
                            <em>There is no data to display...</em>
                        </td>
                    </tr>
                    <tr v-for="(row, key) in results" :key="key" :style="getRowStyling(row)" :class="{'expanded': expandTable}">
                        <td :data-column="col" v-for="col in columns" :key="col" :style="getColumnStyling(row, col)" v-html="displayValue(row[col], col)"></td>
                    </tr>
                </tbody>
                <tfoot v-show="totals" style="position: sticky;bottom: -1px;">
                    <tr class="table-active" v-show="!hideTotals">
                        <th :data-footer="col" v-for="col in columns" :key="col">{{ calculateTotal(col) }}</th>
                    </tr>
                </tfoot>
            </table>
        </div>
        <b-modal :id="`modal-datatable-filter-${_uid}`" scrollable title="Filter Table" size="lg">
            <template #default>
                <div class="form-group" v-for="col in columns" :key="col">
                    <label :for="`filter-${col}`">{{ col | heading }}</label>

                    <div class="input-group" v-if="getFilterOptions(col)">
                        <vue-select v-model.lazy="filterValues[col]" multiple>
                            <option :value="opt" v-for="opt in getFilterOptions(col)" :key="opt">{{ opt }}</option>
                        </vue-select>
                    </div>

                    <div class="input-group" v-else-if="getFieldType(col) == 'Boolean'">
                        <b-form-checkbox v-model="filterValues[col]" switch size="lg">
                            {{ col | heading }}
                        </b-form-checkbox>
                    </div>

                    <div class="input-group" v-else-if="getFieldType(col) == 'Date'">
                        <date-picker mode="range" format="Y-m-d" v-model.lazy="filterValues[col]"></date-picker>
                    </div>

                    <template v-else-if="getFieldType(col) == 'Number'">
                        <div class="input-group mb-5">
                            <input type="number" class="form-control" v-model.lazy="filterValues[col].min" placeholder="Min">
                            <div class="input-group-append">
                                <span class="input-group-text"><i class="fas fa-angle-right"></i></span>
                            </div>
                            <input type="number" class="form-control" v-model.lazy="filterValues[col].max" placeholder="Max">
                            <div class="input-group-append">
                                <button type="submit" class="btn btn-secondary" @click="filterValues[col] = {min: null, max: null}">
                                    <i class="fa fa-times"></i>
                                </button>
                            </div>
                        </div>
                    </template>
                    <div class="input-group" v-else>
                        <input type="text" class="form-control" v-model.lazy="filterValues[col]" :placeholder="`Filter ${col}...`">
                        <div class="input-group-append">
                            <button type="submit" class="btn btn-secondary" @click="filterValues[col] = ''">
                                <i class="fa fa-times"></i>
                            </button>
                        </div>
                    </div>
                </div>
            </template>
            <template #modal-footer="{ ok }">
                <b-button variant="info" @click="clearFilters()">
                    Clear Filters
                </b-button>
                <b-button variant="success" @click="ok()">
                    OK
                </b-button>
            </template>
        </b-modal>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                searchTerm: '',
                busy: true,
                sort: {
                    field: null,
                    desc: false
                },
                cssDefs: {
                    bold: 'font-weight: bold; ',
                    underline: 'text-decoration: underline; ',
                    italics: 'font-style: italic; ',
                    strikethrough: 'text-decoration: line-through; ',
                },
                hiddenFields: [],
                filterValues: {},
                showColumnDropdown: false,
                expandTable: true,
                hideTotals: false,
                fieldTypes: {
                    "INT1": "Number",
                    "INTN": "Number",
                    "BIT": "Boolean",
                    "INT2": "Number",
                    "INT4": "Number",
                    "DATETIM4": "Date",
                    "FLT4": "Number",
                    "FLTN": "Number",
                    "MONEY": "Number",
                    "DATETIME": "Date",
                    "DATETIMN": "Date",
                    "FLT8": "Number",
                    "DECIMAL": "Number",
                    "NUMERIC": "Number",
                    "MONEY4": "Number",
                    "INT8": "String",
                    "IMAGE": "String",
                    "TEXT": "String",
                    "GUIDN": "String",
                    "NTEXT": "String",
                    "BIGVARBIN": "String",
                    "BIGVARCHR": "String",
                    "BIGBinary": "String",
                    "BIGCHAR": "String",
                    "NVARCHAR": "String",
                    "NCHAR": "String",
                    "XML": "String",
                    "TIMEN": "Date",
                    "DATEN": "Date",
                    "DATETIME2N": "Date",
                    "DATETIMEOFFSETN": "Date",
                    "UDTTYPE": "String",
                    "SSVARIANTTYPE": "String"
                }
            }
        },
        props: {
            data: Array,
            tableClass: {
                type: String,
                default: 'table table-hover table-vcenter border',
                required: false
            },
            theadClass: {
                type: String,
                required: false
            },
            identifier: {
                type: String,
                required: false
            },
            stickyHeader: {
                type: Boolean,
                required: false,
                default: false
            },
            conditionalFormatting: {
                type: Array,
                required: false
            },
            reportMeta: {
                type: Array,
                required: false,
                default: () => {
                    return [];
                }
            },
            bordered: {
                type: Boolean,
                required: false,
                default: false
            },
            totals: Boolean,
            search: Boolean,
            responsive: Boolean,
            showEmpty: Boolean
        },
        watch: {
            sort: {
                handler(val, oldVal) {
                    this.busy = true;
                    this.sortTable();
                },
                deep: true
            },
            filterValues: {
                handler() {
                    this.busy = true;
                },
                deep: true
            },
            searchTerm() {
                this.busy = true;
            },
            immutableColumns(val) {
                this.filterValues = {};
                val.forEach(column => {
                    this.$set(this.filterValues, column, '');
                })
            },
            reportMeta: {
                immediate: true,
                handler(val) {
                    this.filterValues = {};
                    val.forEach(column => {
                        if(this.getFieldType(column.colName) == 'Number') {
                            this.$set(this.filterValues, column.colName, {min: null, max: null});
                        } else this.$set(this.filterValues, column.colName, '');
                })
                }
            }
        },
        computed: {
            results() {
                this.$nextTick(() => {
                    setTimeout(() => {
                        this.busy = false;
                    }, 250);
                })
                if(this.searchTerm) return this.filteredData.filter(row => {
                    let values = Object.values(row);
                    return (values.filter((item) => (typeof item == 'string' && item.indexOf(this.searchTerm) > -1)).length > 0);
                })
                return this.filteredData;
            },
            filteredData() {
                return this.data.filter(row => {
                    return this.columns.every(item => {
                        if(this.filterValues[item] === '' || this.filterValues[item] == null || ((Array.isArray(this.filterValues[item]) && this.filterValues[item].length < 1))) return true; //If no filters exist

                        if(this.getFieldType(item) == 'Date') {
                            if(this.filterValues[item].includes('to')) {
                                let dateParts = this.filterValues[item].split(' to ').map(v => Date.parse(v));
                                return (Date.parse(row[item]) <= dateParts[1] && Date.parse(row[item]) >= dateParts[0]);
                            } else return (String(row[item]).indexOf(this.filterValues[item]) > -1);
                        }

                        if(this.getFieldType(item) == 'Number') {
                            let range = this.filterValues[item];
                            if(range.min == null && range.max == null) return true;
                            return (Number(row[item]) >= (range.min || 0) && Number(row[item]) <= (range.max || 999999));
                        }

                        if(Array.isArray(this.filterValues[item])) return (this.filterValues[item].includes(row[item])); // If multi select dropdown

                        return (String(row[item]).indexOf(this.filterValues[item]) > -1); // If text input
                    })
                })
            },
            immutableColumns() {
                if(!this.data || !this.data.length) return [];
                if(this.reportMeta.length > 0) {
                    return this.reportMeta.map(v => v.colName);
                } else return Object.keys(this.data[0]);
            },
            columns() {
                if(!this.data || !this.data.length) return [];
                if(this.reportMeta.length > 0) {
                    return this.reportMeta.map(v => v.colName).filter(v => (!this.hiddenFields.includes(v)));
                } else return Object.keys(this.data[0]).filter(v => (!this.hiddenFields.includes(v)));
            }
        },
        filters: {
            heading(val) {
                return val.replace(/_/g, ' ');
            },
        },
        mounted() {
            // Load config from storage
            if(localStorage.useCondensedReportView != null) {
                this.expandTable = !(localStorage.useCondensedReportView === 'true');
            }

            if(this.identifier && localStorage[`table_${this.identifier}_hidden_columns`] != null) {
                this.hiddenFields = localStorage[`table_${this.identifier}_hidden_columns`].split(',');
            }
        },
        methods: {
            toggleTableView() {
                this.expandTable = !this.expandTable;
                localStorage.useCondensedReportView = !this.expandTable;
            },
            clearFilters() {
                Object.keys(this.filterValues).forEach((key) => {
                    if(this.getFieldType(key) == 'Number') {
                        this.filterValues[key] = {min: null, max: null};
                    } else this.filterValues[key] = '';
                });
            },
            getFilterOptions(col_name) {
                let type = this.getFieldType(col_name);
                if(['Boolean', 'Number', 'Date'].includes(type)) return null;
                let options = [...new Set(this.data.map(item => item[col_name]))];
                if(!options.length || options.length > 500) return null; // Revert to text field as too many options

                return options.sort();
            },
            displayValue(value, col_name) {
                let type = this.getFieldType(col_name);
                switch (type) {
                    case 'Boolean':
                        return (value ? '<i class="fas fa-check text-success"></i>' : '<i class="fas fa-times text-danger"></i>');
                    default:
                        return value;
                }
            },
            getFieldType(col) {
                if(!this.reportMeta || !this.reportMeta.length) return 'String';
                let type = this.reportMeta.find(v => (v.colName == col)).type.type;
                return (this.fieldTypes[type]) ? this.fieldTypes[type] : 'String';
            },
            getRowStyling(row) {
                if(!this.conditionalFormatting) return;
                let conditions = this.conditionalFormatting.filter(v => (v.transformation == 'row-colour'));
                if(!conditions.length) return;

                let css = null;

                conditions.forEach(test => {
                    let test_value = (test.value && test.value.startsWith('[') && test.value.endsWith(']')) ? row[test.value.substring(1, test.value.length - 1)] : (test.value || null);
                    if(this.testCondition(test.condition, row[test.column], test_value)) css = `background-color: ${test.colour};`;
                });

                return css;
            },
            getColumnStyling(row, col) {
                if(!this.conditionalFormatting) return;
                let conditions = this.conditionalFormatting.filter(v => (v.column == col && v.transformation != 'row-colour'));
                if(!conditions.length) return;

                let css = '';

                conditions.forEach(test => {
                    let test_value = (test.value && test.value.startsWith('[') && test.value.endsWith(']')) ? row[test.value.substring(1, test.value.length - 1)] : (test.value || null);
                    if(this.testCondition(test.condition, row[col], test_value)) {
                        if (test.transformation == 'text-colour') {
                            css += `color: ${test.colour}; `;
                        } else if (test.transformation == 'cell-colour') {
                            css += `background-color: ${test.colour}; `;
                        } else css += this.cssDefs[test.transformation];
                    }
                });

                return css;
            },
            testCondition(condition, column_value, test_value) {
                switch (condition) {
                    case 'equal':
                        return (column_value == test_value);
                    case 'not_equal':
                        return (column_value !== test_value);
                    case 'less':
                        return (parseFloat(column_value) < test_value);
                    case 'less_or_equal':
                        return (parseFloat(column_value) <= test_value);
                    case 'greater':
                        return (parseFloat(column_value) > test_value);
                    case 'greater_or_equal':
                        return (parseFloat(column_value) >= test_value);
                    case 'is_null':
                        return (column_value == null);
                    case 'is_not_null':
                        return (column_value !== null);
                    default:
                        return false;
                }
            },
            exportData() {
                if(typeof XLSX === 'undefined') {
                    console.warn('🚨 XLSX (SheetJS/sheetjs) was not found and is required for data exports...');
                    return;
                }
                this.hideTotals = true;
                this.busy = true;
                setTimeout(() => {
                    var elt = document.getElementById('report-output');
                    var book = XLSX.utils.table_to_book(elt, {sheet:"Table Output", dateNF:'dd/mm/yyyy h:mm:ss', cellDates: true, display: true });
                    XLSX.writeFile(book, `report-output_${Date.now()}.xlsx`)
                    this.busy = false;
                    this.hideTotals = false;
                }, 250);
            },
            toggleColumn(col) {
                if(this.hiddenFields.includes(col)) {
                    let index = this.hiddenFields.indexOf(col);
                    if (index > -1) {
                        this.hiddenFields.splice(index, 1);
                    }
                    return;
                }
                this.hiddenFields.push(col);
                if(this.identifier) {
                    localStorage[`table_${this.identifier}_hidden_columns`] = this.hiddenFields;
                }
            },
            sortTable() {
                this.data = this.data.sort((a,b) => (
                    a[this.sort.field] > b[this.sort.field]) ? (this.sort.desc ? -1 : 1) :
                    ((b[this.sort.field] > a[this.sort.field]) ? (this.sort.desc ? 1 : -1) : 0)
                );
            },
            toggleSort(col) {
                if(this.isSorted(col)) return this.sort.desc = !this.sort.desc;
                this.sort.field = col;
                this.sort.desc = false;
                return;
            },
            isSorted(col) {
                return (this.sort.field === col);
            },
            calculateTotal(col) {
                if(!this.results || !this.results.length) return null;
                let vals = this.results.map(row => (row[col]));
                let total = vals.reduce(function(a, b){
                    return a + b;
                }, 0);
                return (typeof(total) === 'number') ? total.toLocaleString(undefined, {minimumFractionDigits: 2, maximumFractionDigits: 2}) : null;
            }
        }
    }
</script>

<style scoped>
    .wrapper {
        width: 100%;
        max-height: 80vh;
        overflow-y: auto;
    }

    .expanded td {
        white-space: nowrap !important;
    }

    thead tr th {
        cursor:pointer;
        -webkit-tap-highlight-color:transparent;
        position: sticky !important;
        top: -1px !important;
    }
</style>
