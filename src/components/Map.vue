<template>
  <div class="map">
    <h3>Карта офиса</h3>

    <div v-if="!isLoading" class="map-root">
      <MapImage ref="mapImage" />

      <TableImage v-show="false" ref="tableImage" />
    </div>
    <div v-else>Loading...</div>
  </div>
</template>

<script>
import * as d3 from "d3";

import "../directives/click-outside";
import { eventBus } from "../main";

import MapImage from "../assets/images/map.svg";
import TableImage from "../assets/images/workPlace.svg";
import tables from "../assets/data/tables.json";
import legend from "../assets/data/legend.json";
import people from "../assets/data/people.json";

export default {
  components: {
    MapImage,
    TableImage,
  },
  data() {
    return {
      isLoading: false,
      mapImage: null,
      gMapImage: null,
      tableImage: null,
      tables: [],
    };
  },
  created() {
    this.tables = tables;
  },
  mounted() {
    this.isLoading = true;

    this.mapImage = d3.select(this.$refs.mapImage);
    this.gMapImage = this.mapImage.select("g");
    this.tableImage = d3.select(this.$refs.tableImage);

    if (this.gMapImage) {
      this.drawTables();
    } else {
      alert("SVG is incorrect");
    }

    this.tablesEventSetup();

    this.isLoading = false;
  },
  methods: {
    drawTables() {
      const svgTablesGroupPlace = this.gMapImage
        .append("g")
        .classed("groupPlaces", true);

      this.tables.map((table) => {
        const targetSeat = svgTablesGroupPlace
          .append("g")
          .attr("transform", `translate(${table.x}, ${table.y}) scale(0.5)`)
          .attr("id", table._id)
          .classed("employer-place", true);

        targetSeat
          .append("g")
          .attr("transform", `rotate(${table.rotate || 0})`)
          .attr("group_id", table.group_id)
          .classed("table", true)
          .html(this.tableImage.html())
          .attr(
            "fill",
            legend.find((item) => item.group_id === table.group_id)?.color ??
              "transparent"
          );
      });
    },
    tablesEventSetup() {
      document.querySelectorAll(".employer-place").forEach((item) => {
        item.addEventListener("click", (event) => {
          const tableId = event.target.parentNode.parentNode.parentNode.id;

          eventBus.$emit(
            "getEmployeeInfo",
            people.find((item) => item.tableId == tableId)
          );
        });
      });
    },
    emitCloseEmployeeCard() {
      eventBus.$emit("getEmployeeInfo", null);
    },
  },
};
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0px;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>
