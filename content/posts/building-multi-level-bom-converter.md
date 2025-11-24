---
title: "Building a Single-Level to Multi-Level BOM"
date: 2025-11-24T20:34:34+07:00
draft: false
ShowBreadCrumbs: true

---

As an MRP Controller, I've seen firsthand how messy data can disrupt a smooth production flow. A common headache, one that many of my colleagues in supply chain will know, is the Bill of Materials (BOM). Specifically, when we receive BOMs in a single-level format. It looks simple, but this format hides complexity and creates unnecessary work for planners and buyers.

For my own work, and to help others, I developed a simple Python-based tool to convert these tricky single-level BOMs into structured multi-level and flattened views. This is not just a technical exercise; it is a practical solution to a persistent operational problem. This post explains the tool, the business case for it, and how similar automation can bring significant value to your manufacturing operations.

### The Problem with Single-Level BOMs in MRP

A single-level BOM only shows the components directly required to make a parent item. It does not show the components of those components. If you have a bicycle (Level 0), the single-level BOM shows you the frame assembly (Level 1) and the wheel assembly (Level 1). To see the spokes and rims inside the wheel assembly, you must look up a separate BOM for it.

For Material Requirements Planning (MRP) systems, this is a big problem, *lah*. An MRP system needs to see every single part, down to the lowest level, to calculate total demand. When fed single-level data, the MRP run can be incorrect. This leads to common issues:

- **Inaccurate Demand Calculation:** The system fails to "explode" the full BOM, so it doesn't order enough of the lower-level components.
- **Inventory Imbalances:** You end up with too much of some parts and, worse, stockouts of critical others, which stops the production line.
- **Wasted Planner Time:** Your planners and buyers must manually trace dependencies level by level, often using spreadsheets. This is slow and a source of many human errors. One small mistake can cause a big problem later.

### How the BOM Converter Tool Works

To solve this, my tool restructures the data into two useful formats: the multi-level BOM and the flattened BOM.

1. **Multi-Level BOM (Indented BOM):** This format shows the full hierarchy. It is like a family tree for your product. Engineers and planners use this view to understand product structure and dependencies. It is essential for engineering changes and troubleshooting.
2. **Flattened BOM (Summarized BOM):** This format provides a simple shopping list. It shows every individual raw material and the total quantity required to produce one finished product. Procurement and kitting teams love this view because it tells them exactly what to buy and what to prepare for a job order without needing to do manual calculations.

The tool itself is a script that reads a simple CSV file with three columns: `Parent_Part`, `Child_Part`, and `Quantity_Per_Parent`. It then processes this data to build the two output formats.

### The Core Logic

The core of the tool uses a recursive algorithm to walk through the BOM levels. This approach, known as a depth-first search, is efficient for traversing tree-like structures like a BOM. Here is a small, simplified piece of the Python code to show the main idea.

This function takes a parent part and recursively finds all its children until it reaches the lowest-level raw materials, calculating total quantities along the way.

```python
def flatten_bom(bom_data, parent_part, required_quantity=1):
    if parent_part not in bom_data:
        # This part is a raw material (base case)
        return {parent_part: required_quantity}

    total_components = {}
    # Iterate through each child of the current parent
    for child, qty_per_parent in bom_data[parent_part].items():
        # Recursively call the function for the child component
        child_components = flatten_bom(
            bom_data,
            child,
            required_quantity * qty_per_parent
        )

        # Aggregate the quantities
        for component, quantity in child_components.items():
            total_components[component] = total_components.get(component, 0) + quantity

    return total_components
```

This logic ensures that every single component is captured and its total required quantity is accurately calculated, which is the foundation for a reliable MRP run.

### Real-World Scenarios

The value of this tool becomes clear in everyday situations.

- **Case Study : Urgent Production Order**
  
  - **Scenario:** A customer placed an urgent, high-volume order for one of our most complex products. The MRP Controller needed to give procurement a total list of all required raw materials by the end of the day.
  - **Action:** The product has over 500 components across 7 levels. A manual explosion would be slow and risky. The planner used the tool to generate a flattened BOM.
  - **Result:** The flattened BOM provided an exact "shopping list" of all 500+ parts and their total quantities in under a minute. The procurement team could immediately begin securing materials, ensuring the production order started on time.

### A Simple Model for Estimating Cost Savings

Beyond preventing delays, a tool like this produces measurable savings, primarily from reducing manual labor. You can estimate your own potential savings with this simple formula.

The table below shows a calculation based on a planner who spends just a few hours a week manually working on BOMs.

| Metric                               | Value      | Source/Assumption           |
| ------------------------------------ | ---------- | --------------------------- |
| Planner Hours Spent per Week on BOMs | 4 hours    | Internal estimate           |
| Planner's Loaded Hourly Rate         | $5         | Assumed (salary + benefits) |
| **Weekly Savings**                   | **$20**    | (4 hours \* $5)             |
| Work Weeks per Year                  | 50 weeks   | Standard assumption         |
| **Annual Labor Savings**             | **$1,000** | ($20 \* 50 weeks)           |

This $1,000 per year is just from time savings for one planner. It does not include the more significant, but harder to quantify, savings from:

- **Reduced Inventory Holding Costs:** By ordering only what you need, you lower your inventory. Industry sources like APICS often estimate that inventory holding costs can be 20% to 30% of the inventory's value per year. I do not have a specific benchmark study for BOM-related savings, but reducing excess stock directly lowers these costs.
- **Fewer Production Stoppages:** Preventing a stockout of a single, cheap component avoids idling an entire production line, which can save thousands of dollars per hour.

### Conclusion

In manufacturing, success depends on data quality. A Bill of Materials is more than a list of parts; it is the blueprint for your entire supply chain. Treating it with the right structure is fundamental. As I found in my own work, a simple, targeted tool can remove manual work, reduce errors, and let your team focus on high-value activities instead of fighting data problems.