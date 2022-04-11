<template>
  <v-container>  <!-- vue components structure for the main page -->
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
          Explore Semantic Segmentation
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
        <v-col cols="1">
          <v-btn
              :loading="upLoading"
              :disabled="upLoading"
              color="primary"
              class="ma-2 white--text"
              fab
              @click="getFile"
              aria-label="Upload new Image"
            >
            <v-icon dark>
              mdi-cloud-upload
            </v-icon>
          </v-btn>
        </v-col>

        <v-col cols="2">
          <v-select
            class="dropindx"
            v-if="plot"
            v-model="selected"
            :items="classItemsFn"
            outlined
            dense
            @change="
              resetValues();
              changeSelect();
            "
            label="Select Class"
          ></v-select>
        </v-col>

        <v-col cols="2">
          <v-select
            v-if="plot"
            v-model="selected2"
            :items="layerItems"
            class="selectLayer"
            outlined
            prepend-inner-icon="mdi-layers"
            hide-spin-buttons
            dense
            @change="changeSelect()"
            label="Select layer"
          >
          </v-select>
        </v-col>
      </v-row>
      <v-row align-content="center" align="center">
        <v-col cols="4" style="vertical-align:middle;">
          <v-container style="text-align:center; padding-left:30%; padding-right:8%; padding-bottom: 10%;">
            <p class="text">Input Image</p>
            <v-img 
              ref="inputImg"
              v-if="url"
              :src="url"
              class="inputImg"
            ></v-img>
          </v-container>
        </v-col>

        <!-- <v-col cols="4">
          <plotly
            v-if="plot"
            :data="data_input"
            :layout="layout_input"
            :displayModeBar="false"
          />
        </v-col> -->
        
        <v-col cols="4">
          <plotly
            v-if="plot"
            :data="data_mask"
            :layout="layout_mask"
            :displayModeBar="false"
            ref="graph"
            id="graph"
            @selected="printval($event)"
          />
        </v-col>
        <v-col cols="4" style="padding-right:40px;">
          <plotly
            v-if="plot_explanation"
            :data="data_explanation"
            :layout="layout_explanation"
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
    upLoading: false,
    plot: false,
    plot_explanation: false,
    classItems: ["background", "cap", "ring", "stipe", "gills", "volva", "mycelium"],
    discrete_colorscale: ['rgb(166,206,227)','rgb(31,120,180)','rgb(178,223,138)','rgb(51,160,44)','rgb(251,154,153)','rgb(227,26,28)','rgb(253,191,111)','rgb(255,127,0)','rgb(202,178,214)','rgb(106,61,154)','rgb(255,255,153)','rgb(177,89,40)'],
    layerItems: [],
    selected: 0,
    selected2: "final_conv", // default value 
    data_input: [{
        z: [],
        type: "heatmap",
        // type: "contour",
        autorange: "reversed",
        colorbar: {
          title: "Class",
          // font: {
          //   family: 'Roboto, normal',
          //   size: 16,
          //   color: '#7f7f7f'
          // },
          tickvals: [0, 1, 2, 3, 4, 5, 6], 
          ticktext: ["background", "cap", "ring", "stipe", "gills", "volva", "mycelium"],  
        },
        // colorscale: "RdYlGn",
        showscale: true,
    }],
    layout_input: {
      title: "Input Image",
      images:[{
        "source": "https://images.plot.ly/language-icons/api-home/r-logo.png",
        "xref": "x",
        "yref": "y",
        "x": 1,
        "y": 3,
        "sizex": 2,
        "sizey": 2,
        "sizing": "stretch",
      // "layer": "below"
      }],
      xaxis: {
        visible: false,
      },
      yaxis: { 
        autorange: "reversed",
        visible: false,
        scaleanchor: "x",
      },
      margin: {
        l: 0,
        r: 0,
        b: 50,
        t: 50,
        pad: 0,
      },
    },
    data_mask: [ //predicted mask 
      {
        z: [],
        type: "heatmap",
        // type: "contour",
        autorange: "reversed",
        colorbar: {
          title: "Class",
          // font: {
          //   family: 'Roboto, normal',
          //   size: 16,
          //   color: '#7f7f7f'
          // },
          tickvals: [0, 1, 2, 3, 4, 5, 6], 
          ticktext: ["background", "cap", "ring", "stipe", "gills", "volva", "mycelium"],  
        },
        // colorscale: "RdYlGn",
        showscale: true,
      },
    ],
    data_explanation: [ // seg-grad-cam
      {
        z: [],
        type: "heatmap",
        autorange: "reversed",
        cmin: 0,
        cmax: 1,
        colorbar: {
          title: "Importance",
          // tickvals: [0, 1], 
          // ticktext: ["low", "high"],
          // orientation: "h" //"v"
        },
        // colorscale: [[0.0,"rgb(255,255,255)"],[1.0,"rgb(165,0,38)"]],
        colorscale: "sequential",
      },
    ], // layout of the heatmaps
    layout_mask: {
      title: "Predicted Mask",
      xaxis: {
        visible: false,
        // fixedrange: true,
      },
      yaxis: { 
        autorange: "reversed",
        visible: false,
        scaleanchor: "x",
        // fixedrange: true,
      },
      tickmode: "array",
      tickvals: [0, 1, 2, 3, 4, 5],
      // width: 450,
      // height: 450,
      margin: {
        l: 0,
        r: 0,
        b: 50,
        t: 50,
        pad: 0,
      },
      // paper_bgcolor: "rgba(27, 27, 50, 0.03)",
      // borderRadius: '20px',
      dragmode: "select",
    },
    layout_explanation: {
      title: "Pixel Importance",
      xaxis: {
        visible: false,
        // fixedrange: true,
      },
      yaxis: { 
        autorange: "reversed",
        visible: false,
        scaleanchor: "x",
        // fixedrange: true,
      },
      // width: 450,
      // height: 450,
      margin: {
        l: 0,
        r: 0,
        b: 50,
        t: 50,
        pad: 0,
      },
      // paper_bgcolor: "rgba(27, 27, 50, 0.03)",
      // borderRadius: '20px',
    },
    imagge: null,
    url: null,
    arrayselected: { x: [], y: [] },
  }),
  computed: {
    classItemsFn() {
      return this.classItems.map((item, index) => {
        return {
          text: item,
          value: index,
        };
      });
    },
    layerItemsFn() {
      return this.layerItems.map((item, index) => {
        return {
          text: item,
          value: index,
        };
      });
    },
  },

  mounted() {
    // this.$refs.graph.$on("click", (d) => {
    //   this.resetValues();
    //   this.printval(d);
    // });
  },

  methods: {
    postForm(e) {
      this.imagge = e.target.files[0];
      this.plot = true;
      this.upLoading = true;
      let formData = new FormData();
      formData.append("image", e.target.files[0]);
      axios
        .post("http://localhost:1025/form-example", formData)
        .then((response) => {
          this.data_mask[0].z = response.data.data;
          this.classItems = response.data.elements;
          this.data_mask[0].colorscale = this.classItemsColorscale();
          this.url = 'data:image/jpeg;base64,' + response.data.inputImg
          this.data_mask[0].colorbar.tickvals = Array.from({length: this.classItems.length},(v,k)=>k)
          this.data_mask[0].colorbar.ticktext = this.classItems
          // this.layout_mask[0].width = this.$refs.inputImg.clientWidth
          // this.layout_mask[0].height = this.$refs.inputImg.clientHeight
          // tickvals: [0, 1, 2, 3, 4, 5, 6], 
          // ticktext: ["background", "cap", "ring", "stipe", "gills", "volva", "mycelium"],  

          this.changeSelect();
          this.getLayers();
          this.plot = true;
          this.upLoading = false;

          this.$refs.graph.$on("click", (d) => {
            this.resetValues();
            this.printval(d);
          });
        }).finally(() => {
          this.upLoading = false;
        });
    }, // for chaning the layer 
    getLayers() {
      let formData = new FormData();
      formData.append("image", this.imagge);
      axios.get("http://localhost:1025/form-example").then((response) => {
        this.layerItems = response.data;
      });
    }, // for chaning selection
    changeSelect() {
      this.plot_explanation = false;
      let self = this;
      let formData = new FormData();
      formData.append("image", this.imagge);
      formData.append("item", this.selected);
      formData.append("item2", this.selected2);
      formData.append("xselected", this.arrayselected.y);
      formData.append("yselected", this.arrayselected.x);

      axios
        .post("http://localhost:1025/change", formData)
        .then((response) => {
          self.data_explanation[0].z = response.data;
          this.plot_explanation = true;
        })
        .finally((res) => {
          console.log(res);
        });
    },
    resetValues() { //clear the values of x and y each time
      this.arrayselected = { x: [], y: [] };
    },
    getFile() {
      this.$refs.uploader.click();
    },
    printval(e) {
      if(e == null)
        return;
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
    classItemsColorscale() {
      const classes_double = this.classItems.reduce(function (res, current) {
          return res.concat([current, current]);
      }, []);
      return classes_double.map((item, index) => {
        const actual_index = this.classItems.indexOf(item);
        if(index % 2 === 1){
          return [(actual_index+1)/this.classItems.length, this.discrete_colorscale[actual_index%this.discrete_colorscale.length]];
        }
        return [actual_index/this.classItems.length, this.discrete_colorscale[actual_index%this.discrete_colorscale.length]];
      })
    }
  },
  watch: {

  }
};
</script>
//styling
<style scoped>
.selectLayer {
  /* width: 187px; */
  /* right: 335px; */
  border-radius: 4px;
  background: #fffdfd;
  flex: none;
  order: 0;
  flex-grow: 1;
  margin: 0px 0px;
}
  
.inputImg{
  /* width:calc(.5*100%); */
  /* height:calc(.5*100%); */
  object-fit: contain;
  border: 2px  #726e6e;
  /* border-radius: 5px; */
}
/* v-img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
} */
.loader{
  width: 450px;
  height: 900px;
}
.dropindx{
  /* width: 187px; */
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
