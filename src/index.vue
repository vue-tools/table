<style src=./style.css></style>

<script>
    import Vue from 'vue'
    import Column from './column'
    
    Vue.component('Column', Column)

    export default {
        name: 'Tables',
        props: {
            index: Boolean,
            selection: Boolean,
            data: {
                type: Array,
                required: true
            }
        },
        data() {
            return {
                columns: []
            }
        },
        render(h) {
            return (
                <table class="ui-table" border="0" cellspacing="0" cellpadding="0">
                    {header(h, this.selection, this.index, this.columns, this.selectionAll, this.$refs)}
                    {body(h, this.selection, this.index, this.data, this.columns, this._renderProxy, this.$vnode.context, this.selectionOne)}
                    <div class="ui-table__slots--hidden" ref="slots">{this.$slots.default}</div>
                    <col width="50" v-show={this.index}></col>
                    <col width="50" v-show={this.selection}></col>
                    {this.columns.map((item) => <col width={item.width}></col>)}
                </table>
            )
        },
        methods: {
            selectionAll(e) {
                Object.keys(this.$refs)
                    .filter((ref) => ref.includes('self'))
                    .map((ref) => this.$refs[ref].checked = e.target.checked)

                this.selection && this.$emit('selection-change', e.target.checked ? this.data : [])
            },
            selectionOne() {
                let refs, result

                result = []
                refs = Object.keys(this.$refs).filter((ref) => ref.includes('self'))

                for(let ref of refs) {
                    if(this.$refs[ref].checked) {
                        result.push(this.data[parseInt(ref.replace('self', ''), 10)])
                    }
                }

                this.$refs.all0.checked = refs.length === result.length
                this.selection && this.$emit('selection-change', result)
            }
        }
    }

    function header(h, selection, index, columns, selectionAll, refs) {
        return (
            <tr class="ui-table__row">
                <td class="ui-table__cell" v-show={selection}>{checkBox(h, 'all0', selectionAll)}</td>
                <td class="ui-table__cell" v-show={index}>#</td>
                {columns.map((item, index) => <td class="ui-table__cell">{item.$options.propsData.title}{sortable(h, item, index, refs)}</td>)}
            </tr>
        )
    }

    function body(h, selection, index, datas, columns, _renderProxy, context, selectionHandler) {
        return datas.map((data, idx) => {
            return (
                <tr class="ui-table__row">
                    <td class="ui-table__cell" v-show={selection}>{checkBox(h, `self${idx}`, selectionHandler)}</td>
                    <td class="ui-table__cell" v-show={index}>{idx + 1}</td>
                    {columns.map((item) => column(h, item, _renderProxy, data, idx, context))}
                </tr>
            )
        })
    }

    function column(h, item, _renderProxy, data, $index, _self) {
        return <td class="ui-table__cell">{item.render.call(_renderProxy, h, {data, $index, _self})}</td>
    }

    function checkBox(h, ref, handler) {
        return (
            <div class="ui-table__checkbox">
                <label class="ui-table__checkbox-label">
                    <input type="checkbox" class="ui-table__checkbox-input" ref={ref} onChange={handler}/>
                    <div class="ui-table__checkbox-checked"></div>
                </label>
            </div>
        )
    }

    function sortable(h, column, index, refs) {
        function handler(index, direction) {
            column.$emit('sort-change', direction, column.$options.propsData.field)
            refs[`topArrow${index}`].classList[direction === 'top' ? 'add' : 'remove']('ui-table__top-arrow--active')
            refs[`bottomArrow${index}`].classList[direction === 'top' ? 'remove' : 'add']('ui-table__bottom-arrow--active')
        }

        if(['string', 'boolean'].includes(typeof column.$options.propsData.sortable)) {
            return (
                <div class="ui-table__sortable">
                    <div class="ui-table__arrow ui-table__top-arrow" ref={`topArrow${index}`} onClick={handler.bind(null, index, 'top')}></div>
                    <div class="ui-table__arrow ui-table__bottom-arrow" ref={`bottomArrow${index}`} onClick={handler.bind(null, index, 'bottom')}>
                    </div>
                </div>
            )
        }
    }
</script>