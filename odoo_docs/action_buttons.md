# Action Buttons

## 1. Feature Overview

**Purpose:** 
[Placeholder: Allow users to trigger specific operations or workflows directly from views (forms, lists, kanban). These can include saving records, changing stages, running wizards, or executing custom Python code.]

**Module Location:** 
[Placeholder: \`web\`, \`base\`, and various specific Odoo modules that define their own actions.]

**Dependencies:** 
[Placeholder: Core view architecture, ORM, \`ir.actions.server\`, \`ir.actions.act_window\`.]

**User Roles:** 
[Placeholder: Availability often depends on user permissions for the action being triggered and the record itself.]

## 2. Technical Details

**Model/View Type:** 
[Placeholder: Defined in XML views (form, tree) using \`<button>\` elements. Actions can be of type \`object\` (call a Python method), \`action\` (trigger an \`ir.actions.act_window\`), or \`workflow\` (legacy, signal a workflow).]

**XML Structure:** 
[Placeholder: \`<button name="method_name" string="Label" type="object" class="oe_highlight"/>\`, \`<button name="%(action_id)d" string="Label" type="action"/>\`, \`attrs="{'invisible': [('state', '=', 'draft')]}"\`, \`confirm="Are you sure?"\`]

**Python Backend:** 
[Placeholder: Python methods on the corresponding model if \`type="object"\`. \`ir.actions.server\` or \`ir.actions.act_window\` records define behavior for \`type="action"\`.]

**Database Impact:** 
[Placeholder: Can range from no impact (e.g., opening a wizard) to creating/updating/deleting records, or changing record state.]

## 3. Visual Documentation

**Mermaid Diagrams:**
\`\`\`mermaid
graph TD
    A[User Clicks Button] --> B{Button Type?};
    B -- Object --> C[Python Method on Model Executed];
    B -- Action --> D[ir.actions.act_window / ir.actions.server Triggered];
    C --> E[View Updates / Response Returned];
    D --> E;
\`\`\`
[Placeholder: Show the flow from button click to action execution and potential UI updates.]

**Screenshots Description:** 
[Placeholder: Show examples of buttons in a form view header (e.g., "Save", "Create", "Confirm") and in a list view (e.g., buttons defined in a list's control panel or row-specific buttons if applicable).]

**UI Layout:** 
[Placeholder: Typically placed in the header of form views (\`<header>\`), or within the sheet. Can also be part of control panels for list/kanban views.]

## 4. Functionality Description

**Core Features:** 
[Placeholder: Execute Python methods on models. Trigger window actions (e.g., open a wizard, another view). Change the state of a record. Provide conditional visibility or availability based on record data or user permissions.]

**Configuration Options:** 
[Placeholder: \`name\` (method or action ID), \`string\` (label), \`type\` (\`object\`, \`action\`), \`class\` (styling, e.g., \`oe_highlight\`, \`oe_link\`), \`attrs\` (dynamic attributes like \`invisible\`, \`readonly\`), \`confirm\` (confirmation dialog), \`icon\` (button icon).]

**Behavior Variations:** 
[Placeholder: Buttons can be highlighted. Can be made invisible or readonly based on conditions. Can trigger server-side logic or client-side actions.]

**Integration Points:** 
[Placeholder: Central to user interaction in most Odoo views. Connects UI events to backend logic or other UI components like wizards and reports.]

## 5. Implementation Examples

**XML Configuration:**
\`\`\`xml
<!-- Provide actual XML view definitions -->
<record id="view_sale_order_form_with_buttons_example" model="ir.ui.view">
    <field name="name">sale.order.form.buttons.example</field>
    <field name="model">sale.order</field>
    <field name="arch" type="xml">
        <form string="Sale Order">
            <header>
                <button name="action_confirm" type="object" string="Confirm" class="oe_highlight"
                        attrs="{'invisible': [('state', 'not in', ['draft', 'sent'])]}"/>
                <button name="action_quotation_send" type="object" string="Send by Email"
                        attrs="{'invisible': [('state', 'not in', ['draft', 'sent'])]}"/>
                <button name="%(sale.action_view_sale_advance_payment_inv)d" string="Create Invoice" type="action"
                        attrs="{'invisible': [('invoice_status', '!=', 'to invoice')]}"/>
                <field name="state" widget="statusbar" statusbar_visible="draft,sent,sale,done"/>
            </header>
            <sheet>
                <!-- other fields -->
                <field name="partner_id"/>
                <field name="order_line"/>
            </sheet>
        </form>
    </field>
</record>
\`\`\`

**Python Code:**
\`\`\`python
# Show related Python model/method implementations
from odoo import models, fields, api

class ExampleSaleOrderButtons(models.Model):
    _inherit = 'sale.order' # Assuming sale.order is the model

    # state = fields.Selection([('draft', 'Draft'), ('sent', 'Sent'), ('sale', 'Sale'), ('done', 'Done')], default='draft')
    # invoice_status = fields.Selection([('no', 'Nothing to Invoice'), ('to invoice', 'To Invoice')], default='no')


    def action_confirm(self):
        self.ensure_one()
        # Logic to confirm the sale order
        self.write({'state': 'sale'})
        return True # Or an action dictionary

    def action_quotation_send(self):
        self.ensure_one()
        # Logic to open the email composer wizard for sending the quotation
        # This often returns an action dictionary to open the wizard
        compose_form_id = self.env.ref('mail.email_compose_message_wizard_form').id
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'views': [(compose_form_id, 'form')],
            'view_id': compose_form_id,
            'target': 'new',
            'context': {
                # context for the wizard
                'default_model': 'sale.order',
                'default_res_id': self.id,
            }
        }
\`\`\`

**JavaScript (if applicable):**
\`\`\`javascript
// Placeholder: JS for very custom client-side button interactions (less common for standard buttons)
// odoo.define('your_module.action_button_customization', function (require) {
// "use strict";
//
// var core = require('web.core');
// var FormController = require('web.FormController');
// var _t = core._t;
//
// FormController.include({
//     _onButtonClicked: function (event) {
//         if (event.data.attrs.name === 'my_custom_js_button') {
//             // client-side only logic
//             alert(_t("Custom JS button clicked!"));
//             return;
//         }
//         this._super.apply(this, arguments);
//     },
// });
// });
\`\`\`

## 6. Customization Guide

**Common Modifications:** 
[Placeholder: Adding new buttons to perform model methods. Changing button labels, classes, or icons. Making buttons conditionally visible/invisible using \`attrs\`. Adding \`confirm\` messages.]

**Extension Points:** 
[Placeholder: Inheriting XML views to add or modify \`<button>\` elements. Adding new Python methods to models to be called by \`type="object"\` buttons. Creating \`ir.actions.server\` or \`ir.actions.act_window\` records to be triggered by \`type="action"\` buttons.]

**Best Practices:** 
[Placeholder: Use \`oe_highlight\` for primary positive actions. Use \`attrs\` for conditional visibility rather than complex logic in Python methods if possible. Keep Python methods concise and focused on a single action.]

**Pitfalls to Avoid:** 
[Placeholder: Overcrowding views with too many buttons. Inconsistent button placement or styling. Buttons that perform destructive actions without confirmation.]

## 7. Testing & Troubleshooting

**Test Scenarios:** 
[Placeholder: 
1. Verify button visibility based on different record states or user roles (using \`attrs\`).
2. Click each button and verify the expected action (Python method execution, wizard opening, state change).
3. Test \`confirm\` messages.
4. Ensure buttons are correctly enabled/disabled based on conditions.]

**Common Issues:** 
[Placeholder: Button not appearing (check \`attrs\`, group memberships). Button doing nothing (check method name, action ID, Python code errors). Permissions errors when clicking a button.]

**Debug Tips:** 
[Placeholder: Activate developer mode to check button attributes and definitions. Use browser developer tools for client-side issues. Add logging or use a debugger in Python methods called by buttons. Check server logs for errors.]
