# Tree View (List View)

## 1. Feature Overview

**Purpose:** 
Odoo Tree Views (often referred to as List Views) are designed to display multiple records of a specific model in a tabular format. They enable users to efficiently scan, sort, filter, and select records for batch operations or to open individual records for detailed viewing in a Form View. They are a fundamental way to browse and manage collections of data.

**Module Location:** 
Core tree view rendering is handled by the \`web\` module. The basic structure and model definitions rely on the \`base\` module.

**Dependencies:** 
Tree views rely on:
*   The Odoo ORM for data fetching (\`search_read\`) and processing.
*   The core view architecture (\`ir.ui.view\`).
*   View inheritance for customization.

**User Roles:** 
Tree views are generally accessible to users who have read access to the underlying data model. The ability to perform actions (like editing or deleting if the tree is editable, or triggering batch actions) depends on further model-specific permissions.

## 2. Technical Details

**Model/View Type:** 
Tree views are defined as records of the \`ir.ui.view\` model with the \`type\` attribute set to \`tree\`.

**XML Structure:** 
The structure of a tree view is defined using XML. Key elements include:
*   \`<tree>\`: The root element. Common attributes:
    *   \`string\`: An optional title for the view (rarely displayed directly).
    *   \`editable="top/bottom"\`: Allows inline creation and/or editing of records directly in the list. \`top\` adds a creation line at the top, \`bottom\` at the bottom.
    *   \`decoration-{$name}="condition"\`: Applies conditional formatting to rows based on Python expressions. Examples: \`decoration-bf\` (bold), \`decoration-it\` (italic), \`decoration-danger\`, \`decoration-success\`, \`decoration-warning\`, \`decoration-info="state == 'draft'"\`, \`decoration-muted\`.
    *   \`default_order="field_name desc/asc"\`: Specifies the default sorting order for the records.
    *   \`multi_edit="1"\`: Enables multi-record editing for selected fields (requires specific field configuration).
    *   \`limit="N"\`: Default number of records to display per page.
    *   \`expand="1"\`: If grouped, expands the first level of groups by default.
    *   \`js_class="my_custom_list_renderer"\`: Specifies a custom JavaScript class (Owl component) for rendering the view, allowing for significant client-side customization.
    *   \`groups_draggable="false"\`: Disables drag-and-drop reordering of groups if the view is grouped.
*   \`<field name="field_name">\`: Defines a column in the tree view. Common attributes:
    *   \`string\`: The column header text (defaults to the field's label in the model).
    *   \`invisible="1"\`: Hides the column by default (can be shown via "Optional Columns").
    *   \`optional="show/hide"\`: If \`show\`, the column is visible by default but can be hidden. If \`hide\`, it's hidden by default but can be shown.
    *   \`widget="widget_name"\`: Uses a specific widget for display (e.g., \`widget="badge"\`, \`widget="handle"\`, \`widget="priority"\`).
    *   \`sum="Total Label"\`: Displays the sum of this column's values in the footer.
    *   \`avg="Average Label"\`: Displays the average of this column's values in the footer.
*   \`<button>\`: Defines a button within a row (less common, but possible). Usually, buttons related to tree views are in the control panel of the action.
*   \`<header>\`: While \`<header>\` is more prominent in Form views, in the context of Tree views, buttons that operate on selected records are typically defined in the associated \`ir.actions.act_window\` or via the \`sidebar\` in older versions, rather than directly within the \`<tree>\`'s header.
*   \`<groupby name="field_name">\`: Can be used within a \`<search>\` view associated with the tree view to define default grouping options, but the \`default_group_by\` attribute on the \`<tree>\` tag is more direct for the view itself if only one default grouping is needed.

**Python Backend:** 
*   Tree views primarily rely on the ORM's \`search_read()\` method to fetch records based on the current domain (from search filters, context, etc.) and requested fields.
*   The \`fields_view_get()\` method on a model can be overridden to dynamically modify the tree view's architecture (e.g., add or remove fields, change attributes) before it's sent to the client.
*   Compute methods (\`fields. タイプ(compute='_compute_method')\`) on the model provide values for calculated fields displayed in the tree view.
*   If inline editing is enabled, the \`write()\` method of the model is called when changes are saved. For \`editable="top"\`, the \`create()\` method is used.

**Database Impact:** 
*   Primarily read operations (SQL \`SELECT\` queries) to display records.
*   If inline editing is used, \`UPDATE\` SQL statements are executed when records are modified.
*   Actions triggered from selected records (e.g., batch updates) will result in corresponding database operations.

## 3. Visual Documentation

**Mermaid Diagrams:**
\`\`\`mermaid
graph TD
    A[User Opens View with Tree] --> B(Call search_read with domain/context);
    B --> C[Data Fetched from Database];
    C --> D[Tree View Renders Rows & Columns];
    D --> E{User Interaction};
    E -- Clicks Column Header --> F[Sort Records by Column];
    F --> D;
    E -- Selects Checkbox(es) --> G[Records Selected for Batch Action];
    E -- Clicks Record --> H[Open Record in Form View];
    E -- Edits Inline (if editable) --> I[Call write/create on Model];
    I --> D;
    E -- Uses Pagination --> B;
\`\`\`

**Screenshots Description:** 
A typical Odoo tree view displays data in a tabular, spreadsheet-like format. Each row corresponds to a record, and each column represents a field from the model.
*   **Column Headers:** Clickable for sorting records by that column (ascending/descending).
*   **Checkboxes:** Optionally displayed at the beginning of each row for selecting multiple records for batch actions.
*   **Content Cells:** Display data from the record's fields. Special widgets can alter their appearance (e.g., badges for status).
*   **Footer:** May display aggregation values (sum, average) for numeric columns if configured.
*   **Pagination Controls:** Located at the bottom right, allowing users to navigate through pages of records if the total number exceeds the view limit.
*   **Control Panel (Search View):** Above the tree view, the search panel provides filters, group-by options, and favorites that affect the records displayed in the tree.

**UI Layout:** 
The layout is tabular. Column widths can sometimes be adjusted by dragging column separators. The view often includes a control panel (defined by the associated search view) above it for filtering and grouping. Pagination controls are typically present at the bottom.

## 4. Functionality Description

**Core Features:** 
*   Display multiple records in a list/table format.
*   Sort records by clicking column headers (ascending/descending).
*   Select one or more records using checkboxes for batch actions (e.g., delete, mass update via server action).
*   Open individual records in a Form view (typically by clicking on a row).
*   Inline editing of fields if \`editable="top/bottom"\` is set, allowing quick modifications without opening the full form view.
*   Trigger actions defined in the control panel (associated with the action view) that operate on selected records.

**Configuration Options:** 
*   \`editable="top/bottom"\`: Enables inline editing or creation.
*   \`decoration-{$name}="condition"\`: Conditionally formats rows (e.g., \`decoration-danger="state == 'cancel'"` makes cancelled records appear in red).
*   \`limit="N"\`: Sets the default number of records per page.
*   \`default_order="field_name desc"\`: Defines the initial sort order.
*   \`optional="show/hide"\` on \`<field>\`: Allows users to show/hide specific columns.
*   \`widget\` attribute on \`<field>\`: Uses specialized widgets for display (e.g., \`badge\`, \`handle\`, \`monetary\`).
*   \`sum\`, \`avg\` attributes on \`<field>\`: For column aggregation in the footer.
*   \`js_class\`: For advanced client-side rendering customization.
*   \`expand="1"\`: To auto-expand first-level groups if a \`default_group_by\` is active.

**Behavior Variations:** 
*   **Read-only vs. Editable:** Tree views can be purely for display or allow inline data modification.
*   **Conditional Formatting:** \`decoration-*\` attributes change row appearance based on record data.
*   **Custom Rendering:** A \`js_class\` can completely change how the tree view is rendered and behaves on the client side.
*   **Drag-and-Drop:** The \`handle\` widget allows for manual reordering of records if the model supports it (e.g., using a \`sequence\` field).

**Integration Points:** 
*   Often serves as the primary view for accessing collections of records, leading to Form views for detailed interaction.
*   Can be embedded within Dashboard views or as part of more complex view layouts.
*   Selections in tree views can be used as input for batch actions, wizards, or reports.
*   Search view filters and groupers directly control the data displayed in the tree view.

## 5. Implementation Examples

**XML Configuration:**

*Simple Partner Tree (existing example):*
\`\`\`xml
<record id="view_partner_tree_example" model="ir.ui.view">
    <field name="name">res.partner.tree.example</field>
    <field name="model">res.partner</field>
    <field name="arch" type="xml">
        <tree string="Partners">
            <field name="name"/>
            <field name="phone"/>
            <field name="email"/>
            <field name="city" optional="hide"/> <!-- Optional column -->
            <field name="country_id"/>
        </tree>
    </field>
</record>
\`\`\`

*More Complex Example (inspired by \`account.move\`'s \`view_invoice_tree\`):*
\`\`\`xml
<record id="view_invoice_tree_example" model="ir.ui.view">
    <field name="name">account.invoice.tree.example</field>
    <field name="model">account.move</field> <!-- Using account.move as an example model -->
    <field name="arch" type="xml">
        <tree string="Invoices" decoration-info="state == 'draft'" decoration-muted="state == 'cancel'" default_order="invoice_date desc, name desc">
            <field name="name" string="Number"/>
            <field name="partner_id" optional="show"/>
            <field name="invoice_date" optional="show"/>
            <field name="date_deadline" optional="hide"/>
            <field name="amount_residual" string="Amount Due" sum="Total Amount Due" widget="monetary" options="{'currency_field': 'currency_id'}"/>
            <field name="amount_total" string="Total" sum="Total Amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
            <field name="currency_id" column_invisible="True"/> <!-- Usually invisible, used by monetary widget -->
            <field name="state" widget="badge" decoration-success="state == 'posted'" decoration-info="state == 'draft'"/>
            <!-- Buttons in tree views are less common directly within rows. 
                 Actions are usually in the control panel (defined in ir.actions.act_window)
                 or context menus. For this example, we'll assume actions are external. -->
        </tree>
    </field>
</record>
\`\`\`
*(Note: \`view_invoice_tree\` itself doesn't have a \`<header>\` with buttons; those actions are part of the \`ir.actions.act_window\` definition that uses this tree view.)*

**Python Code:**
\`\`\`python
# Show related Python model/method implementations
from odoo import models, fields, api

class AccountMoveTreeExample(models.Model):
    _inherit = 'account.move' # Example, actual model is account.move

    # Most fields like name, partner_id, invoice_date, amount_total, state, currency_id
    # are standard fields. Tree views primarily use existing model fields.
    # Compute fields used in the tree view are defined in the model as usual.
    # For example, 'amount_residual' might be a computed field.

    # Example of a method that could be called by a button in an action
    # associated with this tree view (e.g., for selected records).
    @api.model
    def action_example_batch_process(self, invoice_ids):
        invoices = self.browse(invoice_ids)
        for invoice in invoices:
            # Perform some action on each selected invoice
            # invoice.do_something()
            pass
        return True

# Note: The ORM's search_read() is the primary method used for fetching data.
# If inline editing ('editable' attribute) is used, the write() method is called.
\`\`\`

**JavaScript (if applicable):**
\`\`\`javascript
// Placeholder: JS for custom client-side tree view rendering or interactions
// This is typically done by defining a component that extends ListRenderer
// and then referencing it using js_class="my_custom_list_renderer" on the <tree> tag.
//
// odoo.define('your_module.CustomListRenderer', function (require) {
// "use strict";
//
//     const ListRenderer = require('web.ListRenderer');
//
//     const CustomListRenderer = ListRenderer.extend({
//         // Override methods to customize rendering or behavior
//         // For example, to change how a cell is rendered:
//         // _renderBodyCell: function (record, node, colIndex, options) {
//         //    var $cell = this._super.apply(this, arguments);
//         //    if (node.attrs.name === 'my_special_field') {
//         //        // Custom rendering for 'my_special_field'
//         //    }
//         //    return $cell;
//         // },
//     });
//
//     return CustomListRenderer;
// });
\`\`\`

## 6. Customization Guide

**Common Modifications:** 
*   Adding or removing columns (fields).
*   Changing the default sort order (\`default_order\`).
*   Applying conditional formatting to rows using \`decoration-*\` attributes.
*   Enabling inline editing (\`editable="top/bottom"\`).
*   Making less critical columns optional using \`optional="show/hide"\`.
*   Using widgets on fields for specialized display (e.g., \`badge\`, \`progressbar\`, \`handle\`).
*   Adding aggregate functions like \`sum\` or \`avg\` to column footers.

**Extension Points:** 
*   **XML View Inheritance:** Use \`xpath\` to modify existing tree views (add fields, change attributes, modify column order).
*   **Control Panel / Action Buttons:** Add buttons to the control panel of the associated \`ir.actions.act_window\` that uses the tree view. These buttons can trigger server actions or client actions on selected records.
*   **JavaScript (\`js_class\`):** For highly custom rendering or client-side behavior, create a new JavaScript class inheriting from \`ListRenderer\` (or its Owl equivalent) and specify it in the \`js_class\` attribute of the \`<tree>\` tag.
*   **Model Methods:** Define or override Python methods on the model that can be called by actions associated with the tree view.

**Best Practices:** 
*   Limit the number of visible columns by default to ensure readability and avoid horizontal scrolling. Use \`optional="hide"\` for less frequently needed columns.
*   Ensure fields used for sorting or grouping are indexed in the database for better performance on large datasets.
*   Optimize any computed fields displayed in the tree view, as they can impact loading times if not efficient.
*   Use decorations sparingly to highlight important information without cluttering the view.

**Pitfalls to Avoid:** 
*   Displaying too many columns by default, leading to poor user experience and performance.
*   Using complex computed fields that are slow to calculate for every record in the list.
*   Over-reliance on custom JavaScript (\`js_class\`) for simple modifications that could be achieved with standard XML attributes, as this can increase maintenance.
*   Inefficient domains in actions or search filters associated with the tree view, causing slow data loading.

## 7. Testing & Troubleshooting

**Test Scenarios:** 
*   Verify all default columns are displayed correctly with proper labels.
*   Test sorting for each sortable column (ascending and descending).
*   If inline editing is enabled, test creating, editing, and saving records directly in the list.
*   Test conditional formatting (\`decoration-*\` rules) with records in different states.
*   Verify pagination works correctly (navigation, records per page).
*   Test selection of multiple records and any associated batch actions.
*   Check if \`optional="show/hide"\` columns can be toggled correctly.
*   Test any action buttons defined in the control panel that operate on tree view selections.

**Common Issues:** 
*   **Column not appearing:** Check field name in XML, model definition, field permissions (ACLs).
*   **Sorting not working as expected:** Ensure the field type supports sorting; for computed fields, ensure \`store=True\` or appropriate search/group methods are implemented.
*   **Slow loading times:** Too many records, complex computed fields without proper optimization, inefficient domains from search filters, or too many columns being fetched.
*   **Decoration rules not applying:** Check the Python expression syntax in the \`decoration-*\` attribute and ensure the fields used are available.
*   **Inline editing problems:** Ensure the field is writable and the user has permission. Check for JavaScript errors in the console.

**Debug Tips:** 
*   **Developer Mode:** "Edit View: Tree" to inspect and modify the XML structure. Check field names and attributes.
*   **Browser Developer Tools:**
    *   Network tab: Inspect the \`search_read\` RPC call to see the domain, fields requested, and data returned.
    *   Console: Look for JavaScript errors, especially if custom \`js_class\` or widgets are used.
*   **Database Query Analysis:** If loading is slow, enable database query logging in Odoo to see the exact SQL queries being executed and analyze their performance.
*   **Server Logs:** Check for any backend errors if actions or inline editing fail.
