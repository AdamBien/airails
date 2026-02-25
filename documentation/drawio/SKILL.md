---
name: drawio
description: Create draw.io overview diagrams with consistent visual style. Use when asked to create, generate, or edit draw.io diagrams, architecture diagrams, component diagrams, or visual overviews of systems and modules. Triggers on "draw.io", "drawio", "create a diagram", "architecture diagram", "component diagram", or requests to visualize high-level system structure. Not for sequence diagrams or class diagrams.
---

# Draw.io Overview Diagrams

Generate valid draw.io XML and save as `<name>.drawio` using the Write tool.

## Lines

- rounded routing: `edgeStyle=orthogonalEdgeStyle;rounded=1;`
- black stroke: `strokeColor=#000000;`
- arrow at target: `endArrow=block;endFill=1;`
- label without shadow: `shadow=0;` on the label cell

## Style

- all nodes: `rounded=1;whiteSpace=wrap;html=1;shadow=1;`
- corner radius: `arcSize=10;`
- font: `fontColor=#000000;fontSize=13;fontStyle=0;`

## BCE Shape Mapping

| Element | Shape | Fill | Border |
|---|---|---|---|
| Business Component (BC) | rounded rectangle | `#dae8fc` (light blue) | `#6c8ebf` |
| Subsystem container | container with title bar | `#f5f5f5` (light gray) | `#666666` |
| External service | rounded rectangle, dashed | `#fff2cc` (light yellow) | `#d6b656` |
| Boundary layer | rounded rectangle | `#d5e8d4` (light green) | `#82b366` |
| Control layer | rounded rectangle | `#e1d5e7` (light purple) | `#9673a6` |
| Entity layer | rounded rectangle | `#fff2cc` (light yellow) | `#d6b656` |

## Color Palette

- BC blue: fill `#dae8fc`, border `#6c8ebf`
- Subsystem gray: fill `#f5f5f5`, border `#666666`
- External yellow: fill `#fff2cc`, border `#d6b656`
- Boundary green: fill `#d5e8d4`, border `#82b366`
- Control purple: fill `#e1d5e7`, border `#9673a6`
- Entity yellow: fill `#fff2cc`, border `#d6b656`

## Content

- visualize only high-level concepts and modules
- show dependencies between components as directed arrows
- omit implementation details, classes, and methods
- group related components visually
- limit diagram to essential relationships

## Layout

- arrange components in logical flow (top-to-bottom or left-to-right)
- consistent spacing: 40px between nodes, 20px padding inside containers
- avoid crossing lines where possible
- use container shapes (`container=1;`) for subsystem grouping
- default node size: `width=160;height=60;`

## Labels

- use short, descriptive names
- avoid technical jargon unless domain-specific
- label arrows only when relationship type is ambiguous
- black font color for all labels
- do not use italic font
- connection labels must be without shadow

## XML Structure

Minimal skeleton — every `.drawio` file must follow this hierarchy:

```xml
<mxfile>
  <diagram name="Overview" id="diag1">
    <mxGraphModel dx="1024" dy="768" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1169" pageHeight="827" math="0" shadow="0">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <!-- nodes and edges here, parent="1" -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

## Example — Minimal 2-Node Diagram

```xml
<mxfile>
  <diagram name="Overview" id="d1">
    <mxGraphModel dx="1024" dy="768" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1169" pageHeight="827" math="0" shadow="0">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <mxCell id="2" value="Orders" style="rounded=1;whiteSpace=wrap;html=1;shadow=1;arcSize=10;fillColor=#dae8fc;strokeColor=#6c8ebf;fontColor=#000000;fontSize=13;fontStyle=0;" vertex="1" parent="1">
          <mxGeometry x="100" y="200" width="160" height="60" as="geometry"/>
        </mxCell>
        <mxCell id="3" value="Payments" style="rounded=1;whiteSpace=wrap;html=1;shadow=1;arcSize=10;fillColor=#dae8fc;strokeColor=#6c8ebf;fontColor=#000000;fontSize=13;fontStyle=0;" vertex="1" parent="1">
          <mxGeometry x="380" y="200" width="160" height="60" as="geometry"/>
        </mxCell>
        <mxCell id="4" value="charges" style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;strokeColor=#000000;fontColor=#000000;fontSize=11;fontStyle=0;endArrow=block;endFill=1;shadow=0;" edge="1" source="2" target="3" parent="1">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

## File Output

- save as `<name>.drawio` using the Write tool
- see `references/example.drawio` for a complete BCE example
