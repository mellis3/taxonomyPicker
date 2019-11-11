<template>
    <div class="treepicker flex">
        <ProgramsTree
            :programs="programs"
            :filters="filtersApplied"
            :categoryTypesActive="categoryTypesActive"
            v-on:filter="filter( $event )"
            v-on:filterByCategory="filterByCategory( $event )"
            v-on:showAll="showAll()"
            v-on:toggleType="toggleCategoryType( $event )"
        />
        <FilteredPrograms
            :allPrograms="programs"
            :leafNodes="leafNodesOnly"
            :filteredPrograms="filteredPrograms"
            :selectedPrograms="selectedPrograms"
            :filters="filtersApplied"
            :categoryTypesActive="categoryTypesActive"
            :showingOnlySelected="showingOnlySelected"
            v-on:searchUpdate="processSearch( $event )"
            v-on:toggleProgramSelection="toggleProgram( $event )"
            v-on:showSelected="showSelectedPrograms()"
            v-on:hideSelected="showAll()"
        />
    </div>
</template>

<script>

//style
import '../assets/css/treepicker.scss';

/*
    data
    this would be fetched in real life
    downloaded programs.xml that current RES returned, so data should be close to prod data
*/
import * as data from '../data/programs.json';

//components
import ProgramsTree from './Tree';
import FilteredPrograms from './Programs';

export default {
    name: 'TreePicker',
    components: {
        ProgramsTree,
        FilteredPrograms
    },
    /*
        on component creation get data
        should fetch it, but imported here for ease in prototype
    */
    created: function(){
        //console.log( data.ArrayOfProgram.Program );
    },
    data() {
        return {
            programs: data.ArrayOfProgram.Program, //needs to include leaf and non-leaf nodes so we can walk the hierarchy tree
            leafNodesOnly: this.onlyLeafNodes(),
            filteredPrograms: this.onlyLeafNodes(),
            filtersApplied: [],
            categoryTypesActive: [ 'programs', 'indirect' ],
            lastSearch: '',
            selectedPrograms: [],
            showingOnlySelected: false
        }
    },

    methods: {

        /*
            Remove any non-leaf nodes from the program pool
        */
        onlyLeafNodes: function(){
            let onlyLeaves = data.ArrayOfProgram.Program.filter( ( p ) => {
                return p.IsLeaf === 'true';
            } );
            console.log( 'leaf nodes only:', onlyLeaves );
            this.filteredPrograms = onlyLeaves;
            return onlyLeaves;
        },

        /*
            only show items with a ancestor/parent matching the id
        */
        /*filter: function( id ){

            console.log( 'toggle filter id:', id );

            /*const filterIndex = this.filtersApplied.indexOf( id );

            //if filter exists then remove, step up to last applied filter
            if( filterIndex > -1 ){
                console.log( 'filter applied, remove instead' );
                this.filtersApplied.splice(filterIndex , 1);

                //if there are any filters to step back to, use those, otherwise reset to show all
                if( this.filtersApplied.length > 0 ){
                    let previousFilter = this.filtersApplied[ this.filtersApplied.length - 1 ];

                    //previous filter could be program or category
                    this.filteredPrograms = this.leafNodesOnly.filter( ( p ) => {
                        return (
                            (
                                p.Path.includes( previousFilter ) && p.ProjectSponsorId !== previousFilter )
                            ) ||
                            p.Category === previousFilter
                        ;
                    } );
                }
                else{
                    this.filteredPrograms = this.leafNodesOnly;
                }
            }

            //if doesn't exist then apply
            else{
                this.filteredPrograms = this.leafNodesOnly.filter( ( p ) => {
                    return p.Path.includes( id ) && p.ProjectSponsorId !== id;
                } );

                //update applied filters
                this.filtersApplied.push( {
                    level: "1",
                    id: id
                } );
            }


            //determine which level this filter should be
            //0 is category level, then 1 for children, 2 for grandchildren etc
            let level = 1;
            let category = this.filtersApplied.find( ( f ) => {
                return f.level === 0;
            } );

            console.log( "currentFilters:", this.filtersApplied );



        },*/


        filter: function( p ){

            console.log( 'toggle filter id:', p );

            p.level = parseInt( p.level, 10 );

            //determine which level this filter should be
            //0 is category level, then 1 for children, 2 for grandchildren etc
            let level = this.filtersApplied.find( ( f ) => {
                return f.level === p.level;
            } );

            //if filter for this level doesn't exist yet then add it
            if( typeof level === "undefined" ){
                this.filtersApplied.push( {
                    level: p.level,
                    id: p.id
                } );
            }

            //if level filter is the current level then remove all decendents
            else if( level.id === p.id ){
                this.filtersApplied.splice( p.level + 1, 10 );
            }

            //else update it, remove any decendents from the previous item at this level
            else{
                this.filtersApplied.splice( p.level + 1, 10 );
                level.id = p.id;

                this.filteredPrograms = this.leafNodesOnly.filter( ( prog ) => {
                    return prog.Path.includes( p.id ) && prog.ProjectSponsorId !== p.id;
                } );
            }

            //filter programs
            this.filteredPrograms = this.leafNodesOnly.filter( ( prog ) => {
                return prog.Path.includes( p.id ) && prog.ProjectSponsorId !== p.id;
            } );

            console.log( "currentFilters:", this.filtersApplied );
        },


        /*
            show all items for a given category
            Could probably merge this with the normal filter function
        */
        filterByCategory: function( categoryId ){
            console.log( 'filter by category', categoryId );

            this.filtersApplied = [ {
                level: 0,
                id: categoryId
            } ];

            this.filteredPrograms = this.leafNodesOnly.filter( ( p ) => {
                return p.Category === categoryId
            } );


            console.log( "currentFilters:", this.filtersApplied );




            /*let filterStatus = this.filtersApplied.filter( ( f ) => {
                return f === categoryId
            } );*/


            //if category filter is not applied
            //or if it is applied but it's a parent of the deepest filter
            /*if(
                filterStatus.length === 0 ||
                ( filterStatus.length > 0 && this.filtersApplied.length > 1 )
            ){
                this.filteredPrograms = this.leafNodesOnly.filter( ( p ) => {
                    return p.Category === categoryId
                } );

                //update filter list with category level
                let category = this.filtersApplied.find( ( f ) => {
                    return f.level === 0;
                } );
                if( typeof category !== "undefined" ){
                    category.id = categoryId;
                }else{
                    this.filtersApplied.push( {
                        level: 0,
                        id: categoryId
                    } );
                }
            }*/


            //
            /*else if( filterStatus.length > 0 && this.filtersApplied.length > 1 ){
                this.filtersApplied = [ categoryId ];
            }*/

            //if it is the only filter applied
            /*else{
                this.showAll();
            }*/

        },

        /* reset to show all items */
        showAll: function(){
            this.filteredPrograms = this.leafNodesOnly;
            this.filtersApplied = [];
            this.showingOnlySelected = false;
        },

        /*
            Process Search results
            Should further refine any existing filters
        */
        processSearch: function( input ){
            console.log( 'search:', input );

            input = input.toLowerCase();

            //possible choices to match against.
            let programBucket = this.filteredPrograms;

            //if going backwards (deleting characters) then need to change bucket
            if( input.length < this.lastSearch.length ){
                console.log( 'going backwards' );
                programBucket = this.leafNodesOnly;

                if( input.length < 3 ){
                    this.showAll();
                }
            }

            //going forwards
            else{
                //wait until min 3 characters
                if( input.length >= 3 ){

                    //find programs whose names
                    let filterResults = programBucket.filter( ( p ) => {
                        return p.ProgramTitle.toLowerCase().includes( input );
                    });

                    //if authors match
                    //authors are harder because the structure of the data can vary
                    let matchingAuthors = [];
                    for( let p of programBucket ){

                        let managers = this.getProgramManagers( p );
                        for( let m of managers ){
                            if(
                                m.FirstName[ '#text' ].toLowerCase().includes( input ) ||
                                m.LastName[ '#text' ].toLowerCase().includes( input )
                            ){
                                matchingAuthors.push( p );
                            }
                        }
                    }

                    this.filteredPrograms = filterResults.concat( matchingAuthors );
                    console.log( this.filteredPrograms );

                    //reset folder filters
                    //this.filtersApplied = [];
                }
            }

            this.lastSearch = input;
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
            Toggle which category types are visible
        */
        toggleCategoryType: function( categoryType ){
            let index = this.categoryTypesActive.indexOf( categoryType );

            //if not active then add
            if( index < 0 ){
                this.categoryTypesActive.push( categoryType );
            }

            //if active then remove
            else{
                this.categoryTypesActive.splice( index, 1 );
            }
        },


        /*
            Toggle whether a program is selected or not
        */
        toggleProgram: function( id ){
            console.log( 'toggle program:', id );
            let index = this.selectedPrograms.indexOf( id );

            //if already selected then remove
            if( index > -1 ){
                this.selectedPrograms.splice( index, 1 );
            }

            //add to selected list
            else{
                this.selectedPrograms.push( id );
            }
        },


        /*
            Show only selected programs
        */
        showSelectedPrograms: function(){
            console.log( 'show only selected programs' );

            let selected = [];
            for( let p of this.selectedPrograms ){

                //find full program data
                let program = this.leafNodesOnly.filter( ( l ) => {
                    return l.ProjectSponsorId === p;
                } );

                if( program.length > 0 ){
                    selected.push( program[0] );
                }
            }

            this.filteredPrograms = selected;
            this.filtersApplied = [];
            this.showingOnlySelected = true;
        }
    }
}
</script>
