<script>
    export default {
        props: {
            title: String,
            field: String,
            sortable: Boolean,
            width: [Number, String]
        },
        render(h) {},
        beforeCreate() {
            this.data = {}
            this.$index = 0
        },
        methods: {
            checkParent() {
                if(this.$parent.$options.name !== 'Tables') {
                    console.error('Column component must within the Table component')
                    return false
                }

                return true
            }
        },
        created() {
            this.render = (h, data) => {
                if(this.$vnode.data.inlineTemplate) {
                    for(let prop in data._self) {
                        if(!data.hasOwnProperty(prop)) {
                            data[prop] = data._self[prop]
                        }
                    }

                    data._staticTrees = this._staticTrees
                    data.$options.staticRenderFns = this.$options.staticRenderFns
                    
                    return this.$options.render.call(data)
                }else {
                    return data.data[this.field]
                }
            }
        },
        mounted() {
            if(this.checkParent()) {
                this.$parent.columns.push(this)
            }
        }
    }
</script>