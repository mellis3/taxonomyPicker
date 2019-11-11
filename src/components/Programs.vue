<template>

    <div class="treepicker_programs">

        <div class="treepicker_programs_header">
            <h2 class="title">Programs</h2>
            <p class="description">Choose one or more programs from the list below. Use the filters to the left to refine the list.</p>

            <Search
                :filters="filters"
                :count="filteredPrograms.length"
                v-on:searchUpdate="$emit( 'searchUpdate', $event )"
            />

            <div class="treepicker_programs_header_results flex justify-between">
                <div
                    v-if="selectedPrograms.length > 0"
                    class="info currentSelection flex vertical-center"
                >
                    <p v-on:click="$emit( 'showSelected' )">Currently Selected <span class="count">{{ selectedPrograms.length }}</span></p>
                    <Button
                        text="+"
                        type="close"
                        classes="hideSelected"
                        v-on:clickHandler="$emit( 'hideSelected' )"
                        v-if="showingOnlySelected === true"
                    />
                </div>
                <p class="info resultCount">Showing {{ filteredPrograms.length }} of {{ leafNodes.length }}</p>
            </div>
        </div>

        <div class="treepicker_program" v-for="p in filteredPrograms">

            <div
                v-if="checkStatus( p.IsIndirect )"
                v-on:click="selectProgram( p.ProjectSponsorId )"
                :class="`treepicker_program_inner ${ p.IsIndirect === 'true' ? 'indirect' : '' } ${ isSelected( p.ProjectSponsorId ) }`"
                :data-id="p.ProjectSponsorId"
            >

                <div class="treepicker_program_breadcrumbs flex wrap">
                    <div
                        :class="[ 'treepicker_program_breadcrumb', isPartOfFilter( p.Category ) ]"
                    >
                        {{ p.Category.toLowerCase() }}
                    </div>
                    <div
                        v-for="b in buildProgramPath( p.Path )"
                        :class="[ 'treepicker_program_breadcrumb', isPartOfFilter( b.id ) ]"
                    >
                        {{ b.name }}
                    </div>
                </div>

                <h4 class="card_title">
                    {{ p.ProgramTitle }}
                </h4>

                <div class="treepicker_program_managers flex">
                    <p class="treepicker_program_manager" v-for="m in getProgramManagers( p )">

                        <!-- use or create avatar component
                            just using placeholder avatar here -->
                        <span class="avatar">
                            <img src="../assets/img/placeholderAvatar.svg" />
                        </span>

                        <span class="name">
                            {{ m.FirstName['#text'] }} {{ m.LastName['#text'] }}
                        </span>
                    </p>
                </div>

                <Button
                    :text="isSelected( p.ProjectSponsorId ) === 'selected' ? 'Selected' : 'Select'"
                    type="secondary"
                    classes="selectProgram"
                    v-on:clickHandler=""
                />

            </div>
        </div>

    </div>
</template>

<script>

import Search from './Search';
import Button from './Buttons';

export default {
    name: 'FilteredPrograms',
    components: {
        Button,
        Search
    },
    props: {
        allPrograms: {
            type: Array
        },
        leafNodes: {
            type: Array
        },
        categoryTypesActive:{
            type: Array
        },
        filteredPrograms: {
            type: Array,
            required: true
        },
        selectedPrograms: {
            type: Array,
            required: true
        },
        showingOnlySelected: {
            type: Boolean
        },
        filters: Array
    },
    methods: {

        /*
            Build Breadcrumbs to show heirarchy of program
            Provided path is IDs of ancestors ( 195/198/program )
            Need to get names from IDs
        */
        buildProgramPath: function( path ){
            let result = [];
            let ancestors = path.split( "\\" );
            ancestors = ancestors.slice( 0, ancestors.length - 1 );

            //if any ancestors other than category
            if( ancestors.length > 0 ){
                for( let a of ancestors ){
                    let p = this.allPrograms.find( ( p ) => {
                        return p.ProjectSponsorId === a;
                    });
                    result.push( {
                        id: p.ProjectSponsorId,
                        name: p.ProgramTitle
                    } );
                }
            }
            return result;
        },

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
            Check if should show program based on category type
        */
        checkStatus: function( status ){
            let result = true;
            if( status === 'true' ){
                if( this.categoryTypesActive.indexOf( 'indirect' ) < 0 ){
                    result = false;
                }
            }
            else{
                if( this.categoryTypesActive.indexOf( 'programs' ) < 0 ){
                    result = false;
                }
            }
            return result;
        },


        /*
            Select program
            Multiple programs can be selected in some instances so TBD if program picker should auto-close on selection event.
        */
        selectProgram: function( id ){
            this.$emit( 'toggleProgramSelection', id );
        },


        /*
            Check if program is in the selected list
        */
        isSelected: function( id ){
            return this.selectedPrograms.indexOf( id ) > -1 ? 'selected' : '';
        },


        /*
            Check if the breadcrumb is part of the current filter and if so highlight it
        */
        isPartOfFilter: function( crumb ){
            let status = this.filters.find( ( f ) => {
                return f.id === crumb;
            } );
            let result = '';

            if( typeof status !== "undefined" ){
                //determine how far removed from furthest level
                let diff = ( this.filters.length - 1 ) - status.level;
                console.log( diff );
                switch( diff ){
                    case 0:
                        result = 'current';
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
            }

            return result;
        }
    },

    watch: {
        filteredPrograms: function( newPrograms ){
            console.log( 'filtered programs:', newPrograms );
        },
        selectedPrograms: function( newPrograms ){
            console.log( 'selected programs:', newPrograms );
        }
    }
}
</script>
