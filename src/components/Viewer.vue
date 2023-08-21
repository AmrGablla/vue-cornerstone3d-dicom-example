<template>
  <input
    type="file"
    id="selectFile"
    style="margin-bottom: 20px"
    @change="handleFileChange"
  />
  <div id="content"></div>
</template>

<script>
import { RenderingEngine, Enums, metaData } from "@cornerstonejs/core";
import cornerstoneDICOMImageLoader from "@cornerstonejs/dicom-image-loader";
import * as cornerstoneTools from "@cornerstonejs/tools";
import uids from "../uids";
import { initDemo } from "../utils/demo/helpers";
import {
  convertMultiframeImageIds,
  prefetchMetadataInformation,
} from "../utils/demo/helpers/convertMultiframeImageIds";

const {
  PanTool,
  WindowLevelTool,
  StackScrollMouseWheelTool,
  ZoomTool,
  ToolGroupManager,
  Enums: csToolsEnums,
} = cornerstoneTools;
const { ViewportType } = Enums;
const { MouseBindings } = csToolsEnums;
const toolGroupId = "myToolGroup";

export default {
  name: "ViewerComponent",
  data() {
    return {
      toolGroupId: "myToolGroup",
      viewport: null,
      element: null,
    };
  },
  async mounted() {
    const content = document.getElementById("content");

    // image div
    const element = document.createElement("div");
    element.oncontextmenu = (e) => e.preventDefault();
    element.id = "cornerstone-element";
    element.style.width = "500px";
    element.style.height = "500px";

    const div = document.createElement("div");
    div.style.display = "flex";
    div.style.flexDirection = "row";

    const metadata = document.createElement("div");
    metadata.style.marginLeft = "400px";

    div.appendChild(element);
    div.appendChild(metadata);

    this.createMetadataRow("Transfer Syntax", metadata);
    this.createMetadataRow("SOPClassUID", metadata);
    this.createMetadataRow("SOPInstanceUID", metadata);
    this.createMetadataRow("Rows", metadata);
    this.createMetadataRow("Columns", metadata);
    this.createMetadataRow("Spacing", metadata);
    this.createMetadataRow("Direction", metadata);
    this.createMetadataRow("Origin", metadata);
    this.createMetadataRow("Modality", metadata);
    this.createMetadataRow("Pixel Representation", metadata);
    this.createMetadataRow("Bits Allocated", metadata);
    this.createMetadataRow("Bits Stored", metadata);
    this.createMetadataRow("High Bit", metadata);
    this.createMetadataRow("Photometric Interpretation", metadata);
    this.createMetadataRow("Window Width", metadata);
    this.createMetadataRow("Window Center", metadata);

    content.appendChild(div);

    this.element = element;
    const dropZone = document.getElementById("cornerstone-element");
    dropZone.addEventListener("dragover", this.handleDragOver, false);
    dropZone.addEventListener("drop", this.handleFileSelect, false);

    await this.run();
  },
  methods: {
    handleFileChange(e) {
      const file = e.target.files[0];
      const imageId = cornerstoneDICOMImageLoader.wadouri.fileManager.add(file);
      this.loadAndViewImage(imageId);
    },
    createMetadataRow(text, metadata) {
      const row = document.createElement("div");

      const label = document.createElement("span");
      label.innerHTML = `${text}: `;

      const value = document.createElement("span");

      value.id = text.replace(/\s/g, "").toLowerCase();

      row.appendChild(label);
      row.appendChild(value);

      metadata.appendChild(row);
    },
    async run() {
      // Init Cornerstone and related libraries
      await initDemo();

      cornerstoneTools.addTool(PanTool);
      cornerstoneTools.addTool(WindowLevelTool);
      cornerstoneTools.addTool(StackScrollMouseWheelTool);
      cornerstoneTools.addTool(ZoomTool);

      // Define a tool group, which defines how mouse events map to tool commands for
      // Any viewport using the group
      const toolGroup = ToolGroupManager.createToolGroup(toolGroupId);

      // Add tools to the tool group
      toolGroup.addTool(WindowLevelTool.toolName);
      toolGroup.addTool(PanTool.toolName);
      toolGroup.addTool(ZoomTool.toolName);
      toolGroup.addTool(StackScrollMouseWheelTool.toolName);

      // Set the initial state of the tools, here all tools are active and bound to
      // Different mouse inputs
      toolGroup.setToolActive(WindowLevelTool.toolName, {
        bindings: [
          {
            mouseButton: MouseBindings.Primary, // Left Click
          },
        ],
      });
      toolGroup.setToolActive(PanTool.toolName, {
        bindings: [
          {
            mouseButton: MouseBindings.Auxiliary, // Middle Click
          },
        ],
      });
      toolGroup.setToolActive(ZoomTool.toolName, {
        bindings: [
          {
            mouseButton: MouseBindings.Secondary, // Right Click
          },
        ],
      });
      // As the Stack Scroll mouse wheel is a tool using the `mouseWheelCallback`
      // hook instead of mouse buttons, it does not need to assign any mouse button.
      toolGroup.setToolActive(StackScrollMouseWheelTool.toolName);

      // Get Cornerstone imageIds and fetch metadata into RAM

      // Instantiate a rendering engine
      const renderingEngineId = "myRenderingEngine";
      const renderingEngine = new RenderingEngine(renderingEngineId);

      // Create a stack viewport
      const viewportId = "CT_STACK";
      const viewportInput = {
        viewportId,
        type: ViewportType.STACK,
        element: this.element,
        defaultOptions: {
          background: [0.2, 0, 0.2],
        },
      };

      renderingEngine.enableElement(viewportInput);

      // Get the stack viewport that was created
      this.viewport = renderingEngine.getViewport(viewportId);

      toolGroup.addViewport(viewportId, renderingEngineId);
    },
    handleFileSelect(evt) {
      console.log(evt);
      evt.stopPropagation();
      evt.preventDefault();

      // Get the FileList object that contains the list of files that were dropped
      const files = evt.dataTransfer.files;

      // this UI is only built for a single file so just dump the first one
      const file = files[0];
      const imageId = cornerstoneDICOMImageLoader.wadouri.fileManager.add(file);
      this.loadAndViewImage(imageId);
    },
    handleDragOver(evt) {
      evt.stopPropagation();
      evt.preventDefault();
      evt.dataTransfer.dropEffect = "copy"; // Explicitly show this is a copy.
    },
    async loadAndViewImage(imageId) {
      await prefetchMetadataInformation([imageId]);
      const stack = convertMultiframeImageIds([imageId]);
      // Set the stack on the viewport
      this.viewport.setStack(stack).then(() => {
        // Set the VOI of the stack
        // viewport.setProperties({ voiRange: ctVoiRange });
        // Render the image
        this.viewport.render();

        const imageData = this.viewport.getImageData();

        const {
          pixelRepresentation,
          bitsAllocated,
          bitsStored,
          highBit,
          photometricInterpretation,
        } = metaData.get("imagePixelModule", imageId);

        const voiLutModule = metaData.get("voiLutModule", imageId);

        const sopCommonModule = metaData.get("sopCommonModule", imageId);
        const transferSyntax = metaData.get("transferSyntax", imageId);

        document.getElementById("transfersyntax").innerHTML =
          transferSyntax.transferSyntaxUID;
        document.getElementById("sopclassuid").innerHTML = `${
          sopCommonModule.sopClassUID
        } [${uids[sopCommonModule.sopClassUID]}]`;
        document.getElementById("sopinstanceuid").innerHTML =
          sopCommonModule.sopInstanceUID;
        document.getElementById("rows").innerHTML = imageData.dimensions[0];
        document.getElementById("columns").innerHTML = imageData.dimensions[1];
        document.getElementById("spacing").innerHTML =
          imageData.spacing.join("\\");
        document.getElementById("direction").innerHTML = imageData.direction
          .map((x) => Math.round(x * 100) / 100)
          .join(",");

        document.getElementById("origin").innerHTML = imageData.origin
          .map((x) => Math.round(x * 100) / 100)
          .join(",");
        document.getElementById("modality").innerHTML =
          imageData.metadata.Modality;

        document.getElementById("pixelrepresentation").innerHTML =
          pixelRepresentation;
        document.getElementById("bitsallocated").innerHTML = bitsAllocated;
        document.getElementById("bitsstored").innerHTML = bitsStored;
        document.getElementById("highbit").innerHTML = highBit;
        document.getElementById("photometricinterpretation").innerHTML =
          photometricInterpretation;
        document.getElementById("windowcenter").innerHTML =
          voiLutModule.windowCenter;
        document.getElementById("windowwidth").innerHTML =
          voiLutModule.windowWidth;
      });
    },
  },
};
</script>
