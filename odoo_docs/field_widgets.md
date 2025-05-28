# Field Widgets

## 1. Feature Overview

**Purpose:** 
[Placeholder: Control the visual representation and interaction of fields in Odoo views (primarily form, list, and kanban). Widgets can provide specialized UI for different data types or offer enhanced user experiences.]

**Module Location:** 
[Placeholder: \`web\` (core widgets), and various modules can define their own widgets (e.g., \`mail\` for chatter, \`account\` for specific financial field widgets).]

**Dependencies:** 
[Placeholder: View architecture, field types, JavaScript framework (Owl for newer widgets, legacy widgets framework for older ones).]

**User Roles:** 
[Placeholder: Users interact with widgets as part of their normal view usage. Developers choose and configure widgets.]

## 2. Technical Details

**Model/View Type:** 
[Placeholder: Specified using the \`widget\` attribute on \`<field>\` elements in XML view definitions.]

**XML Structure:** 
[Placeholder: \`<field name="my_field" widget="specific_widget_name" options="{'option_key': 'value'}"/>\`. Examples: \`widget="many2many_tags"\`, \`widget="statusbar"\`, \`widget="monetary"\`, \`widget="image"\`.]

**Python Backend:** 
[Placeholder: Python model fields define the data type. Some widgets might have server-side counterparts or rely on specific field attributes (e.g., \`currency_field\` for \`monetary\` widget).]

**Database Impact:** 
[Placeholder: Widgets themselves don't directly impact the database, but they facilitate the display and input of data that is stored in the database according to the field's definition.]

## 3. Visual Documentation

**Mermaid Diagrams:**
\`\`\`mermaid
graph TD
    A[View Renders Field] --> B{Widget Specified?};
    B -- Yes --> C[Custom Widget Renders UI];
    B -- No --> D[Default Widget for Field Type Renders UI];
    C --> E[User Interacts with Widget];
    D --> E;
    E --> F[Data Updated in Model];
\`\`\`
[Placeholder: Illustrate how a field's \`widget\` attribute determines its UI rendering.]

**Screenshots Description:** 
[Placeholder: Show examples of various widgets in Odoo: a \`many2many_tags\` widget, a \`statusbar\` widget, a \`monetary\` field with currency symbol, an \`image\` widget displaying a picture, a \`priority\` (star) widget.]

**UI Layout:** 
[Placeholder: Widgets are rendered in place of standard field inputs/displays within the view's layout (form, list cell, kanban card field).]

## 4. Functionality Description

**Core Features:** 
[Placeholder: Provide custom UI for data entry and display. Enhance user experience for specific field types (e.g., color pickers, progress bars, specialized selectors). Can handle complex data interactions (e.g., signature widget, many2many checkboxes).]

**Configuration Options:** 
[Placeholder: The \`widget\` attribute on the field. The \`options\` attribute can pass a dictionary of parameters to the widget's JavaScript implementation (e.g., \`options="{'currency_field': 'my_currency_id'}"\` for monetary widget).]

**Behavior Variations:** 
[Placeholder: Each widget has its own unique appearance and behavior. Some are for display only, others for input. Behavior can be customized via options.]

**Integration Points:** 
[Placeholder: Directly tied to specific fields in views. Interact with the JavaScript framework for rendering and event handling. Can trigger data changes that are processed by the ORM.]

## 5. Implementation Examples

**XML Configuration:**
\`\`\`xml
<!-- Example of various widgets in a form view -->
<record id="view_my_model_form_with_widgets" model="ir.ui.view">
    <field name="name">my.model.form.widgets</field>
    <field name="model">my.model</field> <!-- Replace with your actual model -->
    <field name="arch" type="xml">
        <form string="My Model with Widgets">
            <sheet>
                <group>
                    <field name="name"/>
                    <field name="priority" widget="priority"/> <!-- Star priority widget -->
                    <field name="user_id" widget="many2one_avatar_user"/> <!-- Avatar for user -->
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/> <!-- Tag list -->
                    <field name="amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="description" widget="html"/> <!-- HTML editor -->
                    <field name="state" widget="statusbar" statusbar_visible="draft,done,cancel"/>
                    <field name="progress" widget="progressbar"/>
                    <field name="signature" widget="signature"/>
                    <field name="image_field" widget="image" options="{'size': [90, 90]}"/>
                </group>
            </sheet>
        </form>
    </field>
</record>
\`\`\`

**Python Code:**
\`\`\`python
# Show related Python model/method implementations
from odoo import models, fields

class MyModelWithWidgets(models.Model):
    _name = 'my.model' # Replace with your actual model name
    _description = 'My Model with Various Widgets'

    name = fields.Char(string="Name")
    priority = fields.Selection([('0', 'Low'), ('1', 'Medium'), ('2', 'High')], string="Priority")
    user_id = fields.Many2one('res.users', string="Assigned User")
    tag_ids = fields.Many2many('my.tag.model', string="Tags") # Replace my.tag.model
    amount = fields.Float(string="Amount")
    currency_id = fields.Many2one('res.currency', string="Currency", 
                                  default=lambda self: self.env.company.currency_id)
    description = fields.Html(string="Description")
    state = fields.Selection([('draft', 'Draft'), ('done', 'Done'), ('cancel', 'Cancelled')], 
                             default='draft', string="Status")
    progress = fields.Integer(string="Progress (%)")
    signature = fields.Binary(string="Signature")
    image_field = fields.Image(string="Image")

class MyTagModel(models.Model):
    _name = 'my.tag.model'
    _description = 'My Tag Model for Widget Example'
    name = fields.Char(string="Tag Name", required=True)
    color = fields.Integer(string="Color Index") # Used by many2many_tags options
\`\`\`

**JavaScript (if applicable):**
\`\`\`javascript
// Example of defining a custom JavaScript (Owl) widget (simplified)
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { CharField } from "@web/views/fields/char/char_field";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

export class MyCustomCharField extends CharField {
    static template = "my_module.MyCustomCharField"; // Reference to QWeb template for the widget
    // Custom logic here
}

MyCustomCharField.props = {
    ...standardFieldProps,
    // any custom props
};

registry.category("fields").add("my_custom_char_widget", MyCustomCharField);

// Associated QWeb template (e.g., in my_module/static/src/xml/my_widgets.xml)
/*
<templates>
    <div t-name="my_module.MyCustomCharField" class="o_field_char">
        <strong>My Custom Widget: </strong>
        <input type="text" class="o_input" t-att-value="props.value" t-on-change="onChange"/>
    </div>
</templates>
*/
\`\`\`

## 6. Customization Guide

**Common Modifications:** 
[Placeholder: Applying existing widgets to fields. Setting options for widgets. Creating entirely new custom widgets using JavaScript (Owl components) for unique UI requirements.]

**Extension Points:** 
[Placeholder: Using the \`widget\` attribute in XML. Defining new JavaScript classes that extend base widget functionality. Registering new widgets in the \`web.field_registry\` (legacy) or \`registry.category("fields")\` (Owl).]

**Best Practices:** 
[Placeholder: Leverage existing widgets where possible. When creating new widgets, follow Odoo's JavaScript module structure and conventions. Ensure widgets are responsive and accessible. Provide clear \`options\` for configurability.]

**Pitfalls to Avoid:** 
[Placeholder: Overriding core widget behavior globally without proper consideration. Creating overly complex widgets that are hard to maintain. Not testing widgets across different browsers or devices. Poor performance in widgets handling large datasets.]

## 7. Testing & Troubleshooting

**Test Scenarios:** 
[Placeholder: 
1. Verify the widget renders correctly for the field in different views (form, list, kanban).
2. Test user interaction with the widget (e.g., data input, selection).
3. Verify that data changes made via the widget are correctly saved.
4. Test widget behavior with different \`options\`.
5. If a custom widget, test its specific functionality thoroughly.]

**Common Issues:** 
[Placeholder: Widget not appearing (check widget name in XML, ensure JS is loaded). Widget options not working (check option keys and values). JavaScript errors in the browser console. Data not saving correctly (check field binding and widget's data handling logic).]

**Debug Tips:** 
[Placeholder: Use browser developer tools to inspect widget HTML structure and CSS. Use JavaScript debugger to step through widget code. Check for errors in Odoo server logs if the widget interacts with the backend. For Owl widgets, use the Owl DevTools browser extension.]
