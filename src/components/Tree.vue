<template>
<div class="treepicker_tree">
    <div class="treepicker_tree_reset" v-on:click="$emit( 'showAll' )"></div>
    <div class="treepicker_tree_header">
        <div class="flex">
            <svg class="icon" width="21px" height="20px" viewBox="0 0 21 20">
                <g fill="#000000" stroke="none">
                    <path d="M7.33462225,10.7566609 L0.218475531,2.34181738 C-0.285346333,1.74604802 0.138127494,0.833333333 0.918369503,0.833333333 L19.2505202,0.833333333 C20.0307622,0.833333333 20.4542361,1.74604802 19.9504142,2.34181738 L12.8342675,10.7566609 L12.8342675,18.2488765 C12.8342675,18.9302661 12.1171939,19.3734419 11.5077406,19.0687152 L7.84131044,17.2355001 C7.53077844,17.0802341 7.33462225,16.7628468 7.33462225,16.4156614 L7.33462225,10.7566609 Z M17.2749526,2.6665484 L2.89393709,2.6665484 L8.95112376,9.82917165 C9.09105695,9.99464265 9.16783733,10.2043411 9.16783733,10.4210482 L9.16783733,15.8491668 L11.0010524,16.7657744 L11.0010524,10.4210482 C11.0010524,10.2043411 11.0778328,9.99464265 11.217766,9.82917165 L17.2749526,2.6665484 Z"></path>
                </g>
            </svg>
            <h2 class="title">Filter</h2>
            <Button text="reset" type="secondary" v-on:clickHandler="$emit( 'showAll' )" />
        </div>


        <p class="label">
            Program Hierarchy
        </p>
        <p class="description">Select a group to filter the programs on the right.<br/>Once selected, groups will expand to show children, which you may select to filter the programs further.</p>
    </div>


    <!-- if showing programs -->
    <div
        v-if="categoryTypesActive.indexOf( 'programs' ) > -1"
        class="treepicker_tree_programs"
    >
        <div
            class="treepicker_tree_category"
            :data-category="c.category"
            v-for="c in categories.programs"
        >
            <Categories
                :id="c.id"
                type="program"
                :category="c.category"
                :programs="c.programs"
                :filters="filters"
                v-on:clickHandler="$emit( 'filter', $event )"
                v-on:filterByCategory="$emit( 'filterByCategory', $event )"
            />
        </div>
    </div>

    <!-- if showing indirects -->
    <div
        v-if="categoryTypesActive.indexOf( 'indirect' ) > -1"
        class="treepicker_tree_indirectPrograms"
    >
        <div
            class="treepicker_tree_category"
            :data-category="c.category"
            v-for="c in categories.indirects"
        >

            <Categories
                :id="c.id"
                type="indirect"
                :category="c.category"
                :programs="c.programs"
                :filters="filters"
                v-on:clickHandler="$emit( 'filter', $event )"
                v-on:filterByCategory="$emit( 'filterByCategory', $event )"
            />
        </div>
    </div>

    <div class="treepicker_tree_header">
        <div class="program_types">
            <p class="label">
                Program Types
            </p>
            <p class="description">Uncheck a type to hide programs of that type on the right.</p>
            <Button
                text="Programs"
                type="primary"
                subtype="checkbox"
                :classes="typeActive( 'programs' )"
                v-on:clickHandler="$emit( 'toggleType', 'programs' )"
            />
            <Button
                text="Indirect"
                type="primary"
                subtype="checkbox"
                :classes="typeActive( 'indirect' )"
                v-on:clickHandler="$emit( 'toggleType', 'indirect' )"
            />
        </div>
    </div>

</div>
</template>

<script>

/*
    Load the folder structure for all Programs
    Build the top level nodes
*/

import Categories from './Categories';
import Button from './Buttons';

export default {
    name: 'ProgramsTree',
    props: {
        categoryTypesActive: {
            type: Array,
            required: true
        },
        filters: {
            type: Array
        },
        programs: {
            type: Array,
            required: true
        }
    },
    components: {
        Categories,
        Button
    },
    data() {
        return {
            categories: this.buildCategories()
        }
    },
    methods: {

        /*
            Group programs by type (programs, indirect) then by category
            Called when component is created
        */
        buildCategories: function(){
            let catNum = 1;

            //separate arrays for each type
            let programs = {};
            let indirects = {};
            let result = null;

            for( let p of this.programs ){
                let bucket = p.IsIndirect === 'true' ? indirects : programs;

                if( !bucket[ p[ 'Category' ] ] ){
                    bucket[ p[ 'Category' ] ] = {
                        id: catNum,
                        category: p.Category,
                        programs: []
                    };
                    catNum++;
                }
                bucket[ p[ 'Category' ] ].programs.push( p );
            }

            console.log({
                programs: programs,
                indirects: indirects
            })

            return {
                programs: programs,
                indirects: indirects
            }

        },

        /*
            If showing type of category (program/indirect)
        */
        typeActive: function( type ){
            return this.categoryTypesActive.indexOf( type ) > -1 ? 'active' : '';
        }


        /*
        convertEntryToDate( e ){
            this.$emit( 'manualDate', e.target.value );
        }
        */
    }
}
</script>
