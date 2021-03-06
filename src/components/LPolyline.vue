<script>
import { onMounted, ref, h, inject } from "vue";
import { remapEvents, propsBinder } from "../utils.js";
import { props, setup as polylineSetup } from "../functions/polyline";

/**
 * Polyline component, lets you add and personalize polylines on the map
 */
export default {
  name: "LPolyline",
  props,
  setup(props, context) {
    const leafletRef = ref({});
    const ready = ref(false);

    const addLayer = inject("addLayer");

    const { options, methods } = polylineSetup(props, leafletRef, context);

    onMounted(async () => {
      const { polyline, DomEvent, setOptions } = await import(
        "leaflet/dist/leaflet-src.esm"
      );

      leafletRef.value = polyline(props.latLngs, options);

      const listeners = remapEvents(context.attrs);
      DomEvent.on(leafletRef.value, listeners);

      propsBinder(methods, leafletRef.value, props, setOptions);

      addLayer({
        ...props,
        ...methods,
        leafletObject: leafletRef.value,
      });
      ready.value = true;
    });
    return { ready };
  },
  render() {
    if (this.ready && this.$slots.default) {
      return h("div", { style: { display: "none" } }, this.$slots.default());
    }
    return null;
  },
};
</script>
