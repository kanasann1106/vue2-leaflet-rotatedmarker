<template>
  <div style="display: none;">
    <slot v-if="ready"></slot>
  </div>
</template>

<script>
import { Icon, Marker, DomEvent } from 'leaflet'
import 'leaflet-rotatedmarker';
import { findRealParent, propsBinder } from 'vue2-leaflet';

const props = {
  draggable: {
    type: Boolean,
    custom: true,
    default: false,
  },
  visible: {
    type: Boolean,
    custom: true,
    default: true,
  },
  latLng: {
    type: [Object, Array],
    custom: true,
  },
  icon: {
    custom: false,
    default: () => new Icon.Default(),
  },
  zIndexOffset: {
    type: Number,
    custom: false,
  },
  rotationAngle:{
    type: Number,
    default: () => 0,
  },
  rotationOrigin: {
    type: String,
    default: 'center center'
  },
  options: {
    type: Object,
    default: () => ({}),
  },
};
export default {
  name: 'LRotatedMarker',
  props: props,
  data() {
    return {
      ready: false,
    }
  },
  mounted() {
    const options = this.options;
    if (this.icon) {
      options.icon = this.icon;
    }
    options.draggable = this.draggable;
    options.rotationAngle = this.rotationAngle?this.rotationAngle:0;
    options.rotationOrigin = this.rotationOrigin;
    this.mapObject = new Marker(this.latLng, options);
    this.mapObject.on('move', (ev) => {
      if (Array.isArray(this.latLng)) {
        this.latLng[0] = ev.latlng.lat;
        this.latLng[1] = ev.latlng.lng;
      } else {
        this.latLng.lat = ev.latlng.lat;
        this.latLng.lng = ev.latlng.lng;
      }
    });
    DomEvent.on(this.mapObject, this.$listeners);
    propsBinder(this, this.mapObject, props);
    this.ready = true;
    this.parentContainer = findRealParent(this.$parent);
    this.parentContainer.addLayer(this, !this.visible);
  },
  beforeDestroy() {
    this.parentContainer.removeLayer(this);
  },
  watch: {
    rotationAngle: {
      handler: function () {
        this.options.rotationAngle = this.rotationAngle
      }
    }
  },
  methods: {
    setDraggable(newVal, oldVal) {
      if (this.mapObject.dragging) {
        newVal ? this.mapObject.dragging.enable() : this.mapObject.dragging.disable();
      }
    },
    setVisible(newVal, oldVal) {
      if (newVal == oldVal) return;
      if (this.mapObject) {
        if (newVal) {
          this.parentContainer.addLayer(this);
        } else {
          this.parentContainer.removeLayer(this);
        }
      }
    },
    setLatLng(newVal) {
      if (newVal == null) {
        return;
      }
      if (this.mapObject) {
        let oldLatLng = this.mapObject.getLatLng();
        let newLatLng = {
          lat: newVal[0] || newVal.lat,
          lng: newVal[1] || newVal.lng,
        };
        if (newLatLng.lat != oldLatLng.lat || newLatLng.lng != oldLatLng.lng) {
          this.mapObject.setLatLng(newLatLng);
        }
      }
    }
  }
};

</script>
