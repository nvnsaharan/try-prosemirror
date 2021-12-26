<template>
  <div id="main-div">
    <div id="editor" class="shadow" ref="editor"></div>
  </div>
</template>

<script>
import { EditorState, Plugin } from "prosemirror-state";
import { EditorView } from "prosemirror-view";
import { Schema, DOMParser } from "prosemirror-model";
import { schema } from "prosemirror-schema-basic";
import { addListNodes } from "prosemirror-schema-list";
import { exampleSetup } from "prosemirror-example-setup";

export default {
  name: "IndexPage",
  mounted() {
    let selectionSizePlugin = new Plugin({
      view(editorView) {
        return new SelectionSizeTooltip(editorView);
      },
      props: {
        handleClickOn(view, event) {
          return console.log("click");
        },
      },
    });

    class SelectionSizeTooltip {
      constructor(view) {
        this.tooltip = document.createElement("div");
        this.tooltip.className = "tooltip";
        this.tooltip.id = 'tool-tip';

        this.Input = document.createElement("input");
        this.Input.className = 'input';
        this.Input.placeholder = 'value'; 
        this.Input.type = 'text'; 
        this.tooltip.append(this.Input);

        const separator = document.createElement("div");
        separator.className = 'separator';
        this.tooltip.append(separator);

        this.button = document.createElement("button");
        this.button.className = 'button';
        this.button.innerText = 'Correct';
        this.tooltip.append(this.button);

        view.dom.parentNode.appendChild(this.tooltip);
        this.update(view, null);
      }

      update(view, lastState) {
        let state = view.state;
        // Don't do anything if the document/selection didn't change
        if (
          lastState &&
          lastState.doc.eq(state.doc) &&
          lastState.selection.eq(state.selection)
        )
          return;

        // Hide the tooltip if the selection is empty
        if (state.selection.empty) {
          this.tooltip.style.display = "none";
          return;
        }

        // Otherwise, reposition it and update its content
        this.tooltip.style.display = "";
        let { from, to } = state.selection;
        const content = state.selection.content();
        const contentToShow = content.content.content[0].textContent;
        // These are in screen coordinates
        let start = view.coordsAtPos(from),
          end = view.coordsAtPos(to);
        // The box in which the tooltip is positioned, to use as base
        let box = this.tooltip.offsetParent.getBoundingClientRect();
        // Find a center-ish x position from the selection endpoints (when
        // crossing lines, end may be more to the left)
        let left = Math.max((start.left + end.left) / 2, start.left + 3);
        this.tooltip.style.left = left - box.left + "px";
        this.tooltip.style.bottom = box.bottom - start.top + "px";
        // this.tooltip.textContent = contentToShow
        this.Input.value = contentToShow;
      }

      destroy() {
        this.tooltip.remove();
      }
    }
    const mySchema = new Schema({
      nodes: addListNodes(schema.spec.nodes, "paragraph block*", "block"),
      marks: schema.spec.marks,
    });

    window.view = new EditorView(this.$refs.editor, {
      state: EditorState.create({
        doc: DOMParser.fromSchema(mySchema).parse(""),
        plugins: exampleSetup({ schema: mySchema }).concat(selectionSizePlugin),
      }),
    });
  },
};
</script>

<style>
html {
  font-size: 16px;
}
#main-div {
  margin: 50px auto;
  width: 700px;
}
#content {
  min-height: 300px;
}
#tool-tip {
  background-color: black;
  width: 18rem;
  height: 2rem;
  border-radius: 5px;
  padding: 7px;
  display: flex;
  justify-content: space-between;
}
.input {
  background-color: transparent;
  outline: none;
  border-radius: 4px;
  color: white;
  border: none;
  width: 12rem;
}
.separator {
  width: 0.5px;
  background-color: gray;
}
.button {
  background-color: #05d090;
  border-radius: 3px;
  padding: 5px 10px;
  border: none;
  cursor: pointer;
  width: 4rem;
}
.shadow {
  box-shadow: 0px 0px 10px 5px rgb(0 140 255 / 22%);
}
</style>
