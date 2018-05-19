<template>
    <div>
        <div>
            <h2>Add marker</h2>
            <label>
                <gmap-autocomplete
                        @place_changed="setPlace">
                </gmap-autocomplete>
                <button @click="addMarker">Add</button>
            </label>
            <br/>
            <label>Marker title:</label>
            <input type="text" v-model="markerTitle"/>
            <br>
            <label>Marker color:</label>
            <select v-model="markerIcon">
                <option v-for="icon in icons" v-bind:value="icon.value">
                    {{ icon.text }}
                </option>
            </select>
        </div>
        <br>
        <gmap-map
                :center="center"
                :zoom="12"
                style="width:100%;  height: 400px;"
        >
            <gmap-marker
                    ref="myMarker"
                    :key="index"
                    v-for="(m, index) in markers"
                    :position="m.position"
                    :draggable="true"
                    :title="m.title"
                    :icon="m.icon"
                    @click="center=m.position"
                    @dragend="markerDragged(m)"
                    @dblclick="destroyMarker(m)"
            ></gmap-marker>
        </gmap-map>
        <span>Doubleclick on marker removes it</span>
        <div v-for="marker in markers">
            {{'Marker: ID ' + marker.store_id + ' LAT ' + marker.position.lat +' LNG '+ marker.position.lng}}
            <input type="text" placeholder="title" v-model="markerTitles[marker.store_id].title"/>
            <button @click="updateMarker(marker)">Update</button>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Map",
        data() {
            return {
                center: {lat: 50.44, lng: 30.45},
                markers: [],
                places: [],
                currentPlace: null,

                id: 0,

                markerTitle: '',
                markerTitles: [],

                markerIcon: 'https://maps.gstatic.com/mapfiles/ms2/micons/red-dot.png',
                markerIcons: [],
                icons: [
                    {text: 'red', value: 'https://maps.gstatic.com/mapfiles/ms2/micons/red-dot.png'},
                    {text: 'blue', value: 'https://maps.gstatic.com/mapfiles/ms2/micons/blue-dot.png'},
                    {text: 'green', value: 'https://maps.gstatic.com/mapfiles/ms2/micons/green-dot.png'}
                ]
            };
        },
        methods: {
            setPlace(place) {
                this.currentPlace = place;
            },
            addMarker() {
                if (this.currentPlace) {
                    const markerLoc = {
                        lat: this.currentPlace.geometry.location.lat(),
                        lng: this.currentPlace.geometry.location.lng()
                    };

                    let id = this.id++
                    let title = this.markerTitle
                    let icon = this.markerIcon

                    let marker = {store_id: id, title: title, position: markerLoc, icon: icon}

                    this.markers.push(marker);
                    this.markerTitles.push({id: id, title: title})
                    this.markerIcons.push({id: id, icon: icon})
                    this.places.push(this.currentPlace);

                    localStorage.setItem(marker.store_id, JSON.stringify(marker))

                    this.center = markerLoc;
                    this.currentPlace = null;
                }
            },
            markerDragged(marker) {
                marker.position.lat = this.$refs.myMarker[marker.store_id].$markerObject.position.lat()
                marker.position.lng = this.$refs.myMarker[marker.store_id].$markerObject.position.lng()
            },
            destroyMarker(marker) {
                this.markers.splice(marker.store_id)
                localStorage.removeItem(marker.store_id)
            },
            updateMarker(marker) {
                let storedMarker = JSON.parse(localStorage.getItem(marker.store_id))
                marker.title = storedMarker.title = this.markerTitles[marker.store_id].title
                localStorage.setItem(marker.store_id, JSON.stringify(marker))
            }
        },
        beforeDestroy() {
            localStorage.clear()
        }
    }
</script>

<style scoped>

</style>