<template>
<div
    v-if="showItem( children ) === true"
    class="treepicker_tree_program"
    :data-id="id"
    :data-programlevel="level"
    :data-children="showChildren"
    :data-active="checkFilterStatus( id )"
>
    <div
        class="childrenToggle"
        v-on:click="toggleChildren"
    >

        <!--
            folder icon
            TO DO: use an SVG sprite for efficiency/reusability
        -->
        <!--<div class="icon">
            <svg width="14px" height="12px" viewBox="0 0 36 33">
                <path d="M0,6 L34,6 C35.1045695,6 36,6.8954305 36,8 L36,31 C36,32.1045695 35.1045695,33 34,33 L2,33 C0.8954305,33 1.3527075e-16,32.1045695 0,31 L0,6 Z"></path>
                <path d="M2,0.75 C1.30964406,0.75 0.75,1.30964406 0.75,2 L0.75,9 C0.75,9.69035594 1.30964406,10.25 2,10.25 L19.1446,10.25 C19.2902907,10.25 19.4348597,10.2245305 19.5717788,10.1747418 C20.2205708,9.93881743 20.5552661,9.22161314 20.3193418,8.57282117 L17.7738873,1.57282117 C17.5942558,1.07883469 17.1247785,0.75 16.5991455,0.75 L2,0.75 Z" stroke-width="2" fill="none"></path>
            </svg>
        </div>-->

        {{ title }}
        <span class="count minimal">{{ calculateChildren() }}</span>
        <div class="treepicker_tree_program_authors">
            <span
                v-for="a in authors"
                class="treepicker_tree_program_author"
            >
                {{ `${ a.FirstName[ '#text' ]} ${ a.LastName[ '#text' ]}` }}
            </span>
        </div>
    </div>

    <Program
        v-if="showChildren"
        v-for="c in children"
        :id="c.ProjectSponsorId"
        :title="c.ProgramTitle"
        :authors="getProgramManagers( c )"
        :children="c.children"
        :level="c.Level"
        :filters="filters"
        v-on:clickHandler="$emit( 'clickHandler', $event )"
    />
</div>
</template>

<script>

/*
    Build the parent and loop through any children
    Recursively call Program as needed for all children

    If no children then don't show, only show parents, not end nodes
*/


export default {
    name: 'Program',
    props: {
        authors: {
            type: Array,
            default: []
        },
        children: Array,
        filters: {
            type: Array
        },
        id: {
            type: String,
            required: true
        },
        level: String,
        title: {
            type: String,
            required: true
        }
    },
    data() {
        return {
            showChildren: false
        }
    },
    methods: {
        showItem( children ){
            return children.length > 0 ? true : false;
        },
        toggleChildren(){
            let id = this.id;
            let level = this.level;
            this.$emit( 'clickHandler', { id: id, level: level } );
        },

        /*
            Children should be the sum of all decendents that are leaf nodes. Don't count non-leaves
        */
        calculateChildren(){
            let count = 0;
            for( let c of this.children ){
                if( c.IsLeaf === 'true' ){
                    count++;
                }
                for( let cc of c.children ){
                    if( cc.IsLeaf === 'true' ){
                        count++;
                    }
                }
            }
            return count;
        },

        /*
            Check if program filter is active
        */
        checkFilterStatus( id ){
            //let status = this.filters.indexOf( id ) > -1 ? true : false;
            //this.showChildren = status;
            //return status;

            let status = this.filters.find( ( f ) => {
                return f.id === id;
            } );


            let result = false;
            if( typeof status !== "undefined" ){

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
                }

            }

            this.showChildren = result;

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
        }

    }
}
</script>
