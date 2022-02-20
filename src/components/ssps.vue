<template>
  <v-container>
    <v-sheet outlined color="primary" rounded>
      <v-card outlined style="min-height: 93vh" elevation="0">
        <v-container>
          <v-file-input
            accept="image/png, image/jpeg, image/bmp"
            placeholder="Pick an avatar"
            prepend-icon=""
            append-icon="mdi-camera"
            label="Image"
            outlined
            v-model="imagge"
            @change="postForm()"
          ></v-file-input>
          <v-row align-content="center">
            <v-col cols="6">
              <v-container>
                <v-img v-if="plotimg" :src="url"   max-height="500" max-width="500"></v-img>
              </v-container>
            </v-col>
            <v-col cols="6">
              <plotly
                v-if="plot"
                :data="data1"
                :layout="layout_2"
                :displayModeBar="false"
                ref="graph"
                id="graph"
                @selected="printval($event)"                
              />
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="6">
              <v-container>
                <v-select
                  v-if="plot"
                  v-model="selected2"
                  :items="items2"
                  class="selectLayer"
                  outlined
                  prepend-inner-icon="mdi-layers"
                  hide-spin-buttons
                  dense
                  @change="changeSelect()"
                >
                </v-select>
                <plotly
                  v-if="plot2"
                  :data="data2"
                  :layout="layout_3"
                  :displayModeBar="false"
                />
              </v-container>
            </v-col>
          </v-row>
          <v-row>
            <v-col class="d-flex" sm="6"> </v-col>

            <v-col class="d-flex" sm="6">
              <v-container>
                <v-row>
                  <v-col>
                    <v-select
                      v-if="plot"
                      v-model="selected"
                      :items="itemss"
                      @change="
                        resetValues();
                        changeSelect();
                      "
                      label="Select type"
                    ></v-select>
                  </v-col>
                  <v-col> </v-col>
                </v-row>
              </v-container>
            </v-col>
          </v-row>
        </v-container>
      </v-card>
    </v-sheet>
  </v-container>
</template>

<script>
import { Plotly } from "vue-plotly";
import axios from "axios";
export default {
  name: "ssps",
  components: {
    Plotly,
  },
  data: () => ({
    plot: true,
    plot2: true,
    plotimg: true,
    items: ["background", "cap", "ring", "stipe", "gills", "volva", "mycelium"],
    items2: [],
    selected: 0,
    selected2: "final_conv",
    data: [
      {
        z: [],
        type: "contour",
        autorange: "reversed",
        showscale: false,
      },
    ],
    data1: [
      {
        z: [],
        type: "contour",
        autorange: "reversed",
        showscale: true,
      },
    ],
    data2: [
      {
        z: [],
        type: "contour",
        autorange: "reversed",
      },
    ],
    layout: {
      title: "Ground Truth",
      yaxis: { autorange: "reversed" },
      width: 500,
      height: 500,
      margin: {
        l: 50,
        r: 50,
        b: 100,
        t: 100,
        pad: 2, 
        
          },
      paper_bgcolor: '#c7c7c7',

    },
    layout_2: {
      title: "Predicted Mask",
      yaxis: { autorange: "reversed" },
      showticklabels: true,
      tickmode: 'array',
      tickvals: [0,1, 2, 3,4,5],
      ticktext: [
        
        "background",
        "cap",
        "ring",
        "stipe",
        "gills",
        "volva",
        "mycelium",
      ],
      width: 500,
      height: 500,
      margin: {
        l: 50,
        r: 50,
        b: 100,
        t: 100,
        pad: 2, 
        
          },
      paper_bgcolor: '#c7c7c7',
      dragmode: "select",
    },
    layout_3: {
      title: "Grad Cam",
      yaxis: { autorange: "reversed" },
      width: 500,
      height: 500,
      margin: {
        l: 50,
        r: 50,
        b: 100,
        t: 100,
        pad: 2,
        
          },
      paper_bgcolor: '#c7c7c7',
    },
    imagge: null,
    url: null,
    arrayselected: { x: [], y: [] },
  }),
  computed: {
    itemss() {
      return this.items.map((item, index) => {
        return {
          text: item,
          value: index,
        };
      });
    },
    itemss2() {
      return this.items2.map((item, index) => {
        return {
          text: item,
          value: index,
        };
      });
    },
  },

  mounted() {
    this.$refs.graph.$on("click", (d) => {
      this.resetValues()
      this.printval(d)

    });
  },

  methods: {
    postForm() {
      console.log("hhere", this.imagge);
      this.url = URL.createObjectURL(this.imagge);
      this.plotimg = true;
      let formData = new FormData();
      formData.append("image", this.imagge);
      axios
        .post("http://localhost:1025/form-example", formData)
        .then((response) => {
          this.data[0].z = response.data;
          this.data1[0].z = response.data;
          this.changeSelect();
          this.getLayers();
          this.plot = true;
        });
    },
    getLayers() {
      console.log("hhere", this.imagge);
      this.url = URL.createObjectURL(this.imagge);
      let formData = new FormData();
      formData.append("image", this.imagge);
      axios.get("http://localhost:1025/form-example").then((response) => {
        this.items2 = response.data;
      });
    },
    changeSelect() {
      this.plot2 = false;
      let self = this;
      this.url = URL.createObjectURL(this.imagge);
      let formData = new FormData();
      formData.append("image", this.imagge);
      formData.append("item", this.selected);
      formData.append("item2", this.selected2);
      formData.append("xselected", this.arrayselected.y);
      formData.append("yselected", this.arrayselected.x);

      axios
        .post("http://localhost:1025/change", formData)
        .then((response) => {
          self.data2[0].z = response.data;
          console.log("data", self.data);
          console.log("data", self.data2);
          this.plot2 = true;
        })
        .finally((res) => {
          console.log(res);
        });
    },
    resetValues() {
      this.arrayselected = { x: [], y: [] };
    },
    printval(e) {
      if (e.range) {
      this.arrayselected = e.range;
      this.changeSelect();
      console.log(e);
      console.log(e.points);
      }
      else {
      console.log('e', e.event.x);
      this.arrayselected.x[0] = e.points[0].x;
      this.arrayselected.y[0]= e.points[0].y;
      this.changeSelect();
      console.log(e);
      console.log(e.points);
      }
    },
  },
};
</script>

<style scoped>
.selectLayer {
  position: absolute;
  width: min-content;
  right: 30px;
  z-index: 3000;
}
</style>