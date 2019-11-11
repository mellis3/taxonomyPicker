<template>
<div class="treepicker_tree_category_inner" :data-active="checkFilterStatus( category )">
    <CategoryHeader
        :category="category"
        :type="type"
        :programsCount="calcLeafPrograms( programs )"
        :filters="filters"
        v-on:clickHandler="toggleFilter()"
    />

    <div class="treepicker_tree_programGroup" v-if="showChildren === true">
        <Program
            v-for="p in groupedPrograms"
            :id="p.ProjectSponsorId"
            :title="p.ProgramTitle"
            :authors="getProgramManagers( p )"
            :children="p.children"
            :level="p.Level"
            :filters="filters"
            v-on:clickHandler="$emit( 'clickHandler', $event )"
        />
    </div>
</div>
</template>

<script>
/*
    Load the folder structure for all Programs
    Build the top level nodes
*/

import CategoryHeader from './CategoryHeader';
import Program from './TreeProgram';

export default {
    name: 'Categories',
    props: {
        category: {
            type: String,
            required: true
        },
        type: {
            type: String
        },
        filters: {
            type: Array
        },
        id: {
            type: Number,
            required: true
        },
        programs: {
            type: Array,
            required: true
        }
    },
    components: {
        CategoryHeader,
        Program
    },
    data() {
        return {
            groupedPrograms: this.groupPrograms(),
            showChildren: false
        }
    },
    methods: {

        /*
            Get manager(s)
            If single manager JSON structure is different, so need to tweak to make for loop work correctly

            This function is used in various places so might be worth adjusting data at a global level so this function isn't needed
        */
        getProgramManagers: function( p ){
            let result = [];

            //check if any managers are defined
            if( typeof p.ProgramManagers !== "undefined" ){

                //sometimes managers are nested
                if( typeof p.ProgramManagers.ProgramManager !== "undefined" ){

                    //all nested ones should be an array, even if single person
                    if( !Array.isArray( p.ProgramManagers.ProgramManager ) ){
                        result = [];
                        result.push( p.ProgramManagers.ProgramManager );
                    }else{
                        result = p.ProgramManagers.ProgramManager;
                    }
                }

                //if not nested it's likely a single person
                else{
                    p.ProgramManagers;
                }
            }
            //console.log( "manager:", result );
            return result;
        },

        /*
            Group Programs into parent with nested children as needed
            Called when component is created

        */
        groupPrograms(){
            let map = {};
            let node;
            let roots = [];
            let i;
            for( i = 0; i < this.programs.length; i += 1 ){
                map[ this.programs[i].ProjectSponsorId ] = i; // initialize the map
                this.programs[i].children = []; // initialize the children
            }
            for (i = 0; i < this.programs.length; i += 1) {
                node = this.programs[i];
                if( typeof node.ParentProjectSponsorId !== "object" ){
                    // if you have dangling branches check that map[node.parentId] exists
                    this.programs[
                        map[ node.ParentProjectSponsorId ]
                    ]
                    .children.push( node );
                } else {
                    roots.push( node );
                }
            }
            return roots;
        },

        /*
            toggle the filter for the category
        */
        toggleFilter: function(){
            //this.showChildren = !this.showChildren;
            this.$emit( 'filterByCategory', this.category );
        },

        /*
            Use the current filters to determine the state of the category
            If category is a current filter then expand and show child nodes.
        */
        checkFilterStatus: function( category ){
            let status = this.filters.find( ( f ) => {
                return f.level === 0;
            } );

            let result = false;
            let childrenStatus = false;
            if( typeof status !== "undefined" ){
                if( status.id === category ){

                    //determine how far removed from furthest level
                    let diff = ( this.filters.length - 1 ) - status.level;
                    switch( diff ){
                        case 0:
                            result = true;
                            break;
                        case 1:
                            result = 'parent';
                            break;
                        case 2:
                            result = 'grandparent';
                            break;
                        case 3:
                            result = 'great-grandparent';
                            break;
                        default:

                    }

                    childrenStatus = true;
                }
            }

            this.showChildren = childrenStatus;


            return result;
        },


        /*

        */
        calcLeafPrograms: function( programs ){
            return programs.filter( ( p ) => {
                return p.IsLeaf === 'true';
            } ).length;
        }

    }
}
</script>
