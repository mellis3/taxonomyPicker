<template>
    <div
        :class="[ 'button ' + classes ]"
        :data-type="type"
        :data-subtype="subtype"
        :data-state="state"
        v-on:click="$emit( 'clickHandler' )"
    >
        <div class="button-inner">
            {{ text }}
        </div>
    </div>
</template>

<script>


export default {
    name: 'Button',
    props: {
        arrow: String,
        classes: {
            tyep: String,
            default: ''
        },
        text: String || Array,
        type: String,
        subtype: String,
        state: String
    }
}
</script>
