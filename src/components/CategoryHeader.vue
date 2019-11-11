<template>
<!--<div
    class="treepicker_tree_category_header"
    v-on:click="$emit( 'clickHandler' )"
    :data-filterActive="checkFilterStatus()"
>-->
<div
    class="treepicker_tree_category_header"
    v-on:click="$emit( 'clickHandler' )"
>
    <div
        class="childrenToggle"
    >
        <span v-if="type === 'indirect'" class="typeFlag">{{ type }}</span>
        {{ makeCapitalized( category ) }}
        <span class="count minimal">{{ programsCount }}</span>
    </div>
</div>
</template>

<script>
/*
    Load the folder structure for all Programs
    Build the top level nodes
*/

import Program from './TreeProgram';

export default {
    name: 'CategoryPrograms',
    props: {
        filters: {
            type: Array
        },
        category: {
            type: String,
            required: true
        },
        type: {
            type: String
        },
        programsCount: {
            type: Number,
            required: true
        }
    },
    methods: {

        /*
            Check if category filter is active
        */
        /*checkFilterStatus(){
            return this.filters.indexOf( this.category ) > -1 ? true : false;
        },*/

        /*
            Capitalize the first letter of each word in the string
            Make sure the remaining portion of the word is lowercase as well
        */
        makeCapitalized: function( str ){
            str = str.split( " " );

            return str.map( ( word ) => {
                return word.charAt(0).toUpperCase() + word.slice( 1 ).toLowerCase();
            } ).join( " " );
        }

    }
}
</script>
