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
        created() {
            this.render = (h, data) => {
                if(this.$vnode.data.inlineTemplate) {
                    return renderInlineTemplate.call(this, h, data)
                }else {
                    return renderField.call(this, h, data)
                }
            }
        },
        mounted() {
            let parent = getComponentByTable(this.$parent)

            if(!parent) {
                console.error('Column component must within the Table component')
                return
            }

            parent.columns.push(this)
        }
    }

    function getComponentByTable(component) {
        if(!component || !component.$el.tagName) {
            return null
        }

        if(component.$el.tagName.toUpperCase() === 'TABLE') {
            return component
        }

        return getComponentByTable(component.$parent)
    }

    function renderField(h, data) {
        return data.data[this.field]
    }

    function renderInlineTemplate(h, data) {
        for(let prop in data._self) {
            if(!data.hasOwnProperty(prop)) {
                data[prop] = data._self[prop]
            }
        }

        data._staticTrees = this._staticTrees
        data.$options.staticRenderFns = this.$options.staticRenderFns
        return this.$options.render.call(data)
    }
</script>