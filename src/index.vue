<style src=./style.css scoped></style>

<script>
    import Vue from 'vue'
    import Column from './column'
    
    Vue.component('Column', Column)

    export default {
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
                columns: [],
                scopedId: []
            }
        },
        render(h) {
            let children = [
                createTrElement(h, arr2obj(this.scopedId), [
                    isShowCheckbox.call(this, h, 'all', 0, this.selectionAll),
                    isShowIndex.call(this, h, '#'),
                    this.columns.map((item, index) => {
                        return createTdElement(h, arr2obj(this.scopedId), [
                            item.$options.propsData.title,
                            isShowSortable.call(this, h, index, arr2obj(this.scopedId), item)
                        ])
                    })
                ]),
                ...this.data.map((data, $index) => {
                    return createTrElement(h, arr2obj(this.scopedId), [
                        isShowCheckbox.call(this, h, 'self', $index, this.selectionOne),
                        isShowIndex.call(this, h, $index + 1),
                        this.columns.map((column) => {
                            return createTdElement(h, arr2obj(this.scopedId), column.render.call(this._renderProxy, h, {
                                data,
                                $index,
                                _self: this.$vnode.context
                            }))
                        })
                    ])
                }),
                h('div', { class: 'table__slots--hidden', ref: 'slots' }, this.$slots.default),
                this.index ? h('col', { attrs: { width: 50 }}) : '',
                this.selection ? h('col', { attrs: { width: 50 }}) : '',
                ...this.columns.map((item) => {
                    return item.width ? h('col', { attrs: { width: item.width }}) : h('col') 
                })
            ]

            return createTableElement(h, children)
        },
        methods: {
            selectionAll() {
                Object
                    .keys(this.$refs)
                    .filter((ref) => ref.indexOf('self') !== -1)
                    .map((ref) => this.$refs[ref].checked = event.target.checked)

                this.selection && this.$emit('selection-change', event.target.checked ? this.data : [])
            },
            selectionOne() {
                let refs, result

                result = []
                refs = Object.keys(this.$refs).filter((ref) => ref.indexOf('self') !== -1)

                for(let ref of refs) {
                    if(this.$refs[ref].checked) {
                        result.push(this.data[parseInt(ref.replace('self', ''), 10)])
                    }
                }

                this.$refs.all0.checked = refs.length === result.length

                this.selection && this.$emit('selection-change', result)
            }
        },
        mounted() {
            this.scopedId = getScopedId(this.$el)
        }
    }

    function getScopedId(element) {
        return Object.keys(element.dataset).filter((key) => key.indexOf('v-') !== -1)
    }

    function isShowSortable(h, index, scopedId, column) {
        let bool, ctx

        ctx = this
        bool = column.$options.propsData.sortable
        
        if(typeof bool === 'string' || typeof bool === 'boolean') {
            return h('div', {
                attrs: scopedId,
                class: 'table__sortable'
            }, [
                h('div', {
                    attrs: scopedId,
                    ref: `topArrow${index}`,
                    class: ['table__arrow', 'table__top-arrow'],
                    on: {
                        click() {
                            ctx.$refs[`topArrow${index}`].classList.add('table__top-arrow--active')
                            ctx.$refs[`bottomArrow${index}`].classList.remove('table__bottom-arrow--active')
                            column.$emit('sort-change', 'top', column.$options.propsData.field)
                        }
                    }
                }),
                h('div', {
                    attrs: scopedId,
                    ref: `bottomArrow${index}`,
                    class: ['table__arrow', 'table__bottom-arrow'],
                    on: {
                        click() {
                            ctx.$refs[`topArrow${index}`].classList.remove('table__top-arrow--active')
                            ctx.$refs[`bottomArrow${index}`].classList.add('table__bottom-arrow--active')
                            column.$emit('sort-change', 'down', column.$options.propsData.field)
                        }
                    }
                })
            ])
        }else {
            return ''
        }
    }

    function isShowCheckbox(h, type, index, callback) {
        if(this.selection) {
            return createTdElement(h, arr2obj(this.scopedId), createCheckboxElement(h, type, index, arr2obj(this.scopedId), callback))
        }else {
            return ''
        }
    }

    function isShowIndex(h, text) {
        return this.index ? createTdElement(h, arr2obj(this.scopedId), text) : ''
    }

    function createTableElement(h, children) {
        return h('table', {
            attrs: {
                bordder: 0,
                cellspacing: 0,
                cellpadding: 0
            },
            class: 'table'
        }, children)
    }

    function createTrElement(h, scopedId, children) {
        return h('tr', {
            attrs: scopedId,
            class: 'table__row'
        }, children)
    }

    function createTdElement(h, scopedId, children) {
        if(!Array.isArray(children)) {
            children = [children]
        }

        return h('td', {
            attrs: scopedId,
            class: 'table__cell'
        }, children)
    }

    function createCheckboxElement(h, type, index, scopedId, callback) {
        return h('div', {
            attrs: { ...scopedId },
            class: 'table__checkbox'
        }, [
            h('label', {
                attrs: { ...scopedId },
                class: 'table__checkbox-label'
            }, [
                h('input', {
                    ref: `${type}${index}`,
                    on: { change: callback },
                    class: 'table__checkbox-input',
                    attrs: { ...scopedId, type: 'checkbox' }
                }),
                h('div', {
                    attrs: { ...scopedId },
                    class: 'table__checkbox-checked'
                })
            ])
        ]) 
    }

    function arr2obj(arr, obj = {}) {
        arr.map((item) => {
            obj[`data-${item}`] = ''
        })

        return obj
    }
</script>