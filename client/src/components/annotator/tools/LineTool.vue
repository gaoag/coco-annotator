<script>
import paper from "paper";
import tool from "@/mixins/toolBar/tool";
import UndoAction from "@/undo";

import { invertColor } from "@/libs/colors";
import { mapMutations } from "vuex";

export default {
  name: "LineTool",
  mixins: [tool],
  props: {
    scale: {
      type: Number,
      default: 1
    },
    settings: {
      type: [Object, null],
      default: null
    }
  },
  data() {
    return {
      icon: "fa-pencil",
      name: "Line",
      scaleFactor: 3,
      cursor: "copy",
      line: {
        path: null,
        guidance: true,
        simplify: 1,
        pathOptions: {
          strokeColor: "black",
          strokeWidth: 1
        }
      },
      color: {
        blackOrWhite: true,
        auto: true,
        radius: 10,
        circle: null
      },
      actionTypes: Object.freeze({
        ADD_POINTS: "Added Points",
        FINISHED_LINE: "Finished Line",
        DELETE_LINE: "Delete Line"
      }),
      actionPoints: 0
    };
  },
  methods: {
    ...mapMutations(["addUndo", "removeUndos"]),
    export() {
      return {
        guidance: this.line.guidance,
        blackOrWhite: this.color.blackOrWhite,
        auto: this.color.auto,
        radius: this.color.radius
      };
    },
    setPreferences(pref) {
      this.line.guidance = pref.guidance || this.line.guidance;
      this.color.blackOrWhite = pref.blackOrWhite || this.color.blackOrWhite;
      this.color.auto = pref.auto || this.color.auto;
      this.color.radius = pref.radius || this.color.radius;
    },
    /**
     * Creates a new selection polygon path
     */
    createLine() {
      if (this.color.auto) {
        this.color.circle = new paper.Path.Circle(
          new paper.Point(0, 0),
          this.color.radius
        );
      }
      this.line.path = new paper.Path(this.line.pathOptions);
    },
    /**
     * Frees current line
     */
    deleteLine() {
      if (this.line.path == null) return;
      console.log("this gettin called")
      this.line.path.remove();
      this.line.path = null;

      if (this.color.circle == null) return;
      this.color.circle.remove();
      this.color.circle = null;
    },
    autoStrokeColor(point) {
      if (this.color.circle == null) return;
      if (this.line.path == null) return;
      if (!this.color.auto) return;

      this.color.circle.position = point;
      let raster = this.$parent.image.raster;
      let color = raster.getAverageColor(this.color.circle);
      if (color) {
        this.line.pathOptions.strokeColor = invertColor(
          color.toCSS(true),
          this.color.blackOrWhite
        );
      }
    },
    onMouseDrag(event) {
      if (this.line.path == null) return;
      this.actionPoints++;
      this.autoStrokeColor(event.point);
      this.line.path.add(event.point);
    },
    onMouseDown(event) {
      let wasNull = false;
      if (this.line.path == null) {
        wasNull = true;
        this.createLine();
      }

      this.actionPoints = 1;
      this.line.path.add(event.point);

      if (this.line.guidance && wasNull) this.line.path.add(event.point);
    },
    onMouseUp() {
      if (this.line.path == null) return;
      let action = new UndoAction({
        name: this.name,
        action: this.actionTypes.ADD_POINTS,
        func: this.undoPoints,
        args: {
          points: this.actionPoints
        }
      });

      this.addUndo(action);
    },

    onMouseMove(event) {
      if (this.line.path == null) return;
      if (this.line.path.segments.length === 0) return;
      this.autoStrokeColor(event.point);

      if (!this.line.guidance) return;
      this.removeLastPoint();
      this.line.path.add(event.point);
    },
    /**
     * Undo points
     */
    undoPoints(args) {
      if (this.line.path == null) return;

      let points = args.points;
      let length = this.line.path.segments.length;

      if (this.line.guidance) {
        length -= 1;
      }

      this.line.path.removeSegments(length - points, length);
    },
    
    /**
     * Ends current line and unites it with current annotaiton.
     * @returns {boolean} sucessfully closes object
     */

    complete() {
      if (this.line.path == null) return false;
      console.log("attempt at completion");
      this.removeLastPoint();

      this.line.path.fillColor = "black";

      // // convert the line segments in this path to a polygon

      let reconstructedPath = new paper.Path(this.line.pathOptions);
      // console.log(this.line.path.segments);

      let newPoints1 = [];
      let newPoints2 = [];

      // let offset = this.line.path.length / 2
      // const normal = this.line.path.getNormalAt(offset);
      // console.log(normal);
      // // for (let [key, segment] of Object.entries(this.line.path.segments)){
      // //   const offset = this.line.path.getOffsetOf(segment.point) + 20;
      // //   const normal1 = this.line.path.getNormalAt(offset)*20;
      // //   const normal2 = this.line.path.getNormalAt(offset)*-20;
      // //   // let normalV1 = normal*20;
      // //   // let normalV2 = normal*-20;
      // //   console.log(normal1);
      // //   newPoints1.push(segment.point + normal1);
      // //   newPoints2.push(segment.point + normal2);
      // //   console.log(segment.point.x, segment.point.y);
      // //   console.log(segment.point + normal1);
      // // }

      for (var i = 0; i < this.line.path.segments.length-1; i++) {
        let pt1 = this.line.path.segments[i].point;
        let pt2 = this.line.path.segments[i+1].point;
        // calculate normal vector at midpoint of pt1 and pt2
        const tempPath = new paper.Path(pt1, pt2)
        let normal = tempPath.getNormalAt(tempPath.length / 2);
        // apply that normal vector to the current point
        newPoints1.push(new Point(pt1.x + normal.x, pt1.y + normal.y));
        newPoints2.push(new Point(pt1.x - normal.x, pt1.y - normal.y));

        if (i == this.line.path.segments.length-2) {
          newPoints1.push(new Point(pt2.x + normal.x, pt2.y + normal.y));
        newPoints2.push(new Point(pt2.x - normal.x, pt2.y - normal.y));

        }
      } 


      reconstructedPath.addSegments(newPoints1);
      reconstructedPath.addSegments(newPoints2.reverse());
      reconstructedPath.closePath();

      this.$parent.uniteCurrentAnnotation(reconstructedPath);

      this.line.path.remove();
      this.line.path = null;
      if (this.color.circle) {
        this.color.circle.remove();
        this.color.circle = null;
      }


      this.removeUndos(this.actionTypes.ADD_POINTS);
      return true;
    },
    removeLastPoint() {
      this.line.path.removeSegment(this.line.path.segments.length - 1);
    }
  },
  computed: {
    isDisabled() {
      return this.$parent.current.annotation === -1;
    }
  },
  watch: {
    isActive(active) {
      if (active) {
        this.tool.activate();
        localStorage.setItem("editorTool", this.name);
      }
    },
    /**
     * Change width of stroke based on zoom of image
     */
    scale(newScale) {
      this.line.pathOptions.strokeWidth = newScale * this.scaleFactor;
      if (this.line.path != null)
        this.line.path.strokeWidth = newScale * this.scaleFactor;
    },
    /**
     * Remove last point (point were mouse was) if enable guidane
     */
    "line.guidance"(guidance) {
      if (this.line.path == null) return;

      if (!guidance && this.line.path.length > 1) {
        this.removeLastPoint();
      }
    },
    "line.pathOptions.strokeColor"(newColor) {
      if (this.line.path == null) return;

      this.line.path.strokeColor = newColor;
    },
    "color.auto"(value) {
      if (value && this.line.path) {
        this.color.circle = new paper.Path.Circle(
          new paper.Point(0, 0),
          this.color.radius
        );
      }
      if (!value && this.color.circle) {
        this.color.circle.remove();
        this.color.circle = null;
      }
    }
  },
  created() {},
  mounted() {
  }
};
</script>
