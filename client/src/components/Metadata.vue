<template>
  <div>
    <i
      class="fa fa-plus"
      style="float: right; margin: 0 4px; color: green"
      @click="createMetadata"
    />

    <p class="title" style="margin: 0">{{ title }}</p>

    <div class="row">
      <div class="col-sm">
        <p class="subtitle">{{ keyTitle }}</p>
      </div>
      <div class="col-sm">
        <p class="subtitle">{{ valueTitle }}</p>
      </div>
    </div>

    <ul class="list-group" style="height: 50%;">
      <li v-if="metadataList.length == 0" class="list-group-item meta-item">
        <i class="subtitle">No items in metadata.</i>
      </li>
      <li
        v-for="(object, index) in metadataList"
        :key="index"
        class="list-group-item meta-item"
      >
        <div class="row" style="cell">
          <div class="col-sm">
            <!-- <input
              v-model="object.key"
              type="text"
              class="meta-input"
              :placeholder="keyTitle"
            /> -->

            <select name="metadata-types" 
                v-model="object.key"
                class="meta-input"
                :placeholder="keyTitle">
              <option v-for="metadataType in pairs" :key="metadataType.id" :value="metadataType">
                {{metadataType.label}}</option>
            </select>
              <!-- <option value="landtype">Land Type</option>
              <option value="roadtype">Road Type</option>
              <option value="buildingtype">Building</option>
            </select> -->
          </div>

          <div class="col-sm">
            <select name="metadata-values"
                v-model="object.value"
                class="meta-input"
                :placeholder="valueTitle">
                <!-- <option v-for="option in keyValPairs[object.key].options">{{ option }}</option> -->
                <option v-for="option in object.key.options" :key="option.id">
                  {{ option }}
                </option>
  
            </select>

          </div>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "Metadata",
  props: {
    metadata: {
      type: Object,
      required: true
    },
    title: {
      type: String,
      default: "Metadata"
    },
    keyTitle: {
      type: String,
      default: "Keys"
    },
    valueTitle: {
      type: String,
      default: "Values"
    },
    exclude: {
      type: String,
      default: ""
    },
    validMetadataEntries:
    {
      type: Array,
      default: () => {
        return [
          {
            label:"Land Type",
            options:["forest", "residential", "farm"]
          },
          {
            label:"Building Type",
            options:["home", "factory", "other"]
          },
          {
            label:"Road Type",
            options:["paved", "dirt", "big", "small"]
          }
        ]
      }
    },
  },
  data() {
    return {
      metadataList: [],
      pairs:[
      {
        label:"Land Type",
        options:["forest", "residential", "farm"]
      },
      {
        label:"Building Type",
        options:["home", "factory", "other"]
      },
      {
        label:"Road Type",
        options:["paved", "dirt", "big", "small"]
      }
    ],
    
    selectedDrink:-1,
    selectedOption:''
  
    };
  },
  methods: {
    export() {
      let metadata = {};

      this.metadataList.forEach(object => {
        if (object.key.length > 0) {
          if (!isNaN(object.value))
            metadata[object.key] = parseInt(object.value);
          else if (
            object.value.toLowerCase() === "true" ||
            object.value.toLowerCase() === "false"
          )
            metadata[object.key] = object.value.toLowerCase() === "true";
          else metadata[object.key] = object.value;
        }
      });

      return metadata;
    },

    // appropriateValue(key, value) {
    //   keyValPairs = {
    //     "landtype": ["forest", "farm", "residential"],
    //     "roadtype": ["paved", "dirt", "small", "big"],
    //     "Building": ["home", "factory"]
    //   }
    //   return (keyValPairs[key].includes(value))
    // },

    createMetadata() {
      this.metadataList.push({ key: "", value: "" });
    },
    loadMetadata() {
      // this "watches" instances of new metadata
      if (this.metadata != null) {
        for (var key in this.metadata) {
          if (!this.metadata.hasOwnProperty(key)) continue;
          if (key === this.exclude) continue;

          let value = this.metadata[key];
          // null out values for which the value is "invalid"

          if (value == null) value = "";
          else value = value.toString();

          this.metadataList.push({ key: key, value: value });
        }
      }
    },
    // displayMetadataValues() {

    //   //programatically add in the desired elems, and filter out any elems which don't apply

    //   // use jquery/raw javascript? how does this interact with the vue component?
    // }
  },
  watch: {
    metadata() {
      // this.displayMetadataValues();
      this.loadMetadata();
    }
  },
  created() {
    this.loadMetadata();
  },
  

};
</script>

<style scoped>
.meta-input {
  padding: 3px;
  background-color: inherit;
  width: 100%;
  height: 100%;
  border: none;
}

.meta-item {
  background-color: inherit;
  height: 30px;
  border: none;
}

.subtitle {
  margin: 0;
  font-size: 10px;
}
</style>
