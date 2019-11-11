<template>
    <div class="treepicker_search">
        <div class="move_label_on_focus">
            <input
                class="treepicker_search_value"
                required
                v-on:input="$emit( 'searchUpdate', $event.target.value )"
                ref="searchInput"
            />
            <label>Search by Program Name or Key Person</label>
        </div>
        <div class="icon">
            <svg width="17px" height="19px" viewBox="0 0 20 22" >
                <g stroke-width="2" fill="none" stroke-linecap="round" transform="translate(1, 1)">
                    <path d="M14,15 L18,20"></path>
                    <path d="M16,8 C16,12.4184371 12.4180864,16 7.99975443,16 C3.58142248,16 0,12.4184371 0,8 C0,3.58156293 3.58142248,0 7.99975443,0 C12.4180864,0 16,3.58156293 16,8 L16,8 Z"></path>
                </g>
            </svg>
        </div>
    </div>
</template>

<script>

export default {
    name: 'Search',
    props: {
        count: Number,
        filters: Array
    },
    mounted: function(){
        this.$refs['searchInput'].focus();
    },
    watch: {
        filters: function( newFilters ){
            if( newFilters.length > 0 ){
                this.$refs['searchInput'].value = "";
            }
        }
    }
}
</script>
