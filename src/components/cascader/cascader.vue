<template>
    <div :class="classes" v-clickoutside="handleClose">
        <i-input
            readonly
            :disabled="disabled"
            :value.sync="displayRender"
            :size="size"
            :placeholder="placeholder"
            @on-focus="onFocus"></i-input>
        <Icon type="ios-close" :class="[prefixCls + '-arrow']" v-show="showCloseIcon" @click.stop="clearSelect"></Icon>
        <Icon type="arrow-down-b" :class="[prefixCls + '-arrow']"></Icon>
        <Dropdown v-show="visible" transition="slide-up">
            <div>
                <Caspanel
                    v-ref:caspanel
                    :prefix-cls="prefixCls"
                    :data.sync="data"
                    :disabled="disabled"
                    :change-on-select="changeOnSelect"
                    :trigger="trigger"></Caspanel>
            </div>
        </Dropdown>
    </div>
</template>
<script>
    import iInput from '../input/input.vue';
    import Dropdown from '../select/dropdown.vue';
    import Icon from '../icon/icon.vue';
    import Caspanel from './caspanel.vue';
    import clickoutside from '../../directives/clickoutside';
    import { oneOf } from '../../utils/assist';

    const prefixCls = 'ivu-cascader';

    export default {
        components: { iInput, Dropdown, Icon, Caspanel },
        directives: { clickoutside },
        props: {
            data: {
                type: Array,
                default () {
                    return []
                }
            },
            value: {
                type: Array,
                default () {
                    return []
                }
            },
            disabled: {
                type: Boolean,
                default: false
            },
            clearable: {
                type: Boolean,
                default: true
            },
            placeholder: {
                type: String,
                default: '请选择'
            },
            size: {
                validator (value) {
                    return oneOf(value, ['small', 'large']);
                }
            },
            trigger: {
                validator (value) {
                    return oneOf(value, ['click', 'hover']);
                },
                default: 'click'
            },
            changeOnSelect: {
                type: Boolean,
                default: false
            },
            renderFormat: {
                type: Function,
                default (label) {
                    return label.join(' / ');
                }
            }
        },
        data () {
            return {
                prefixCls: prefixCls,
                visible: false,
                selected: [],
                tmpSelected: []
            }
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}`,
                    {
                        [`${prefixCls}-show-clear`]: this.showCloseIcon,
                        [`${prefixCls}-visible`]: this.visible,
                        [`${prefixCls}-disabled`]: this.disabled
                    }
                ]
            },
            showCloseIcon () {
                return this.value && this.value.length && this.clearable;
            },
            displayRender () {
                let label = [];
                for (let i = 0; i < this.selected.length; i++) {
                    label.push(this.selected[i].label);
                }

                return this.renderFormat(label, this.selected);
            }
        },
        methods: {
            clearSelect () {
                const oldVal = JSON.stringify(this.value);
                this.value = this.selected = this.tmpSelected = [];
                this.handleClose();
                this.emitValue(this.value, oldVal);
                this.$broadcast('on-clear');
            },
            handleClose () {
                this.visible = false;
            },
            onFocus () {
                this.visible = true;
                if (!this.value.length) {
                    this.$broadcast('on-clear');
                }
            },
            updateResult (result) {
                this.tmpSelected = result;
            },
            updateSelected (init = false) {
                if (!this.changeOnSelect || init) {
                    this.$broadcast('on-find-selected', this.value);
                }
            },
            emitValue (val, oldVal) {
                if (JSON.stringify(val) !== oldVal) {
                    this.$emit('on-change', this.value, JSON.parse(JSON.stringify(this.selected)));
                }
            }
        },
        ready () {
            this.updateSelected(true);
        },
        events: {
            // lastValue: is click the final val
            // fromInit: is this emit from update value
            'on-result-change' (lastValue, changeOnSelect, fromInit) {
                if (lastValue || changeOnSelect) {
                    const oldVal = JSON.stringify(this.value);
                    this.selected = this.tmpSelected;

                    let newVal = [];
                    this.selected.forEach((item) => {
                        newVal.push(item.value);
                    });

                    if (!fromInit) {
                        this.value = newVal;
                        this.emitValue(this.value, oldVal);
                    }
                }
                if (lastValue && !fromInit) {
                    this.handleClose();
                }
            }
        },
        watch: {
            visible (val) {
                if (val) {
                    if (this.value.length) {
                        this.updateSelected();
                    }
                }
            },
            value () {
                this.updateSelected();
            }
        }
    }
</script>