<template>
  <v-container>
    <div v-show="!plot">
      <input
        ref="uploader"
        class="d-none"
        type="file"
        accept="image/*"
        @change="postForm"
      />
      <div>
        <h2 style="margin-top: 40vh; text-align: center">
          Easily analyze Semantic Segmentation
        </h2>
        <v-btn
          color="primary"
          style="margin: 0 auto; display: block; margin-top: 50px"
          x-large
          @click="getFile"
        >
          <v-icon left dark> mdi-plus </v-icon> Upload image
        </v-btn>
      </div>
    </div>
    <div outlined v-show="plot" class="mt-8" color="primary" rounded>
      <v-row align-content="center">
        <v-col cols="6">
          <v-container>
            <p class="text">Your Uploaded Image </p>
            <v-img 
              v-if="plotimg"
              :src="url"
              class="input"
            ></v-img>
          </v-container>
        </v-col>
        <v-col cols="6">
          <span
            style="width: 70% margin: 0 auto; display: block; font-size: 16px; margin-top: 120px"
          >
            SSPS: A Web-Based Tool for exploring interpretation of semantic segmentation.
          </span>

          <v-btn
            color="primary"
            style="margin: 0 auto; display: block; margin-top: 50px; margin-left: 150px"
            x-large
            @click="getFile"
          >
            <v-icon left dark> mdi-plus </v-icon> Upload a new image
          </v-btn>
        </v-col>
        <v-col cols="6">
          <v-row>
            <v-col cols="8">
              <v-select
                class="dropindx"
                v-if="plot"
                v-model="selected"
                :items="itemss"
                outlined
                dense
                @change="
                  resetValues();
                  changeSelect();
                "
                label="Select Class index"
              ></v-select>
            </v-col>
            <v-col cols="4">
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
                label="Select Final layer"
              >
              </v-select>
            </v-col>
          </v-row>
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
        <v-col style="margin-top: 68px" cols="6">
          <plotly
            v-if="plot2"
            :data="data2"
            :layout="layout_3"
            :displayModeBar="false"
          />
          <v-skeleton-loader
            class="loader"
            type="image"
            v-else
          ></v-skeleton-loader>
        </v-col>
      </v-row>
      <v-row> </v-row>
    </div>
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
    plot: false,
    plot2: false,
    plotimg: false,
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
    layout_2: {
      title: "Predicted mask",
      yaxis: { autorange: "reversed" },
      showticklabels: true,
      tickmode: "array",
      tickvals: [0, 1, 2, 3, 4, 5],
      ticktext: [
        "background",
        "cap",
        "ring",
        "stipe",
        "gills",
        "volva",
        "mycelium",
      ],
      width: 450,
      height: 450,
      margin: {
        l: 50,
        r: 50,
        b: 50,
        t: 50,
        pad: 2,
      },
      paper_bgcolor: "rgba(27, 27, 50, 0.03)",
      borderRadius: '20px',
      dragmode: "select",
    },
    layout_3: {
      title: "Segmentation-grad-cam",
      yaxis: { autorange: "reversed" },
      width: 450,
      height: 450,
      margin: {
        l: 50,
        r: 50,
        b: 50,
        t: 50,
        pad: 2,
      },
      paper_bgcolor: "rgba(27, 27, 50, 0.03)",
      borderRadius: '20px',
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
      this.resetValues();
      this.printval(d);
    });
  },

  methods: {
    postForm(e) {
      console.log("hhere", e.target.files[0]);
      this.imagge = e.target.files[0];
      this.url = URL.createObjectURL(e.target.files[0]);
      this.plotimg = true;
      let formData = new FormData();
      this.plot = true;
      formData.append("image", e.target.files[0]);
      axios
        .post("http://localhost:1025/form-example", formData)
        .then((response) => {
          this.data[0].z = response.data.data;
          this.data1[0].z = response.data.data;
          this.items= response.data.elements
          this.changeSelect();
          this.getLayers();
          this.plot = true;
          this.$refs.graph.$on("click", (d) => {
          this.resetValues();
          this.printval(d);
    });
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
    getFile() {
      this.$refs.uploader.click();
    },
    printval(e) {
      if (e.range) {
        this.arrayselected = e.range;
        this.changeSelect();
        console.log(e);
        console.log(e.points);
      } else {
        console.log("e", e.event.x);
        this.arrayselected.x[0] = e.points[0].x;
        this.arrayselected.y[0] = e.points[0].y;
        this.changeSelect();
        console.log(e);
        console.log(e.points);
      }
    },
  },
  watch: {

  }
};
</script>

<style scoped>
.selectLayer {
  width: 187px;
  right: 335px;
  border-radius: 4px;
  background: #fffdfd;
  flex: none;
  order: 0;
  flex-grow: 1;
  margin: 0px 0px;
  
  }
  
.input{
width:calc(.5*100%);
height:calc(.5*100%);
object-fit: contain;
border: 2px  #726e6e;
border-radius: 5px;
}
v-img {
 max-width: 100%;
 max-height: 100%;
  object-fit: contain;

}
.loader{
width: 450px;
height: 900px;
}
.dropindx{
  width: 187px;
  background: #fffdfd; 
  border-radius: 4px;
  flex: none;
  order: 0;
  flex-grow: 1;
  margin: 0px 0px;

}
.text{
font-family: Roboto;
font-style: normal;
font-weight: 400;
font-size: 16px;
line-height: 24px;
}

</style>
