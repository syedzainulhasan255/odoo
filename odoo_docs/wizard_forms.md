# Wizard Forms

## 1. Feature Overview

**Purpose:** 
[Placeholder: Guide users through a multi-step process or collect specific input for an operation that doesn't require a permanent record (or creates/updates records upon completion). Often used for complex operations, configurations, or generating reports.]

**Module Location:** 
[Placeholder: \`base\`, and various modules defining specific wizards.]

**Dependencies:** 
[Placeholder: \`ir.ui.view\`, \`models.TransientModel\` (typically), \`ir.actions.act_window\`.]

**User Roles:** 
[Placeholder: Depends on the action that launches the wizard and the underlying operations.]

## 2. Technical Details

**Model/View Type:** 
[Placeholder: Models are typically \`models.TransientModel\`. Views are standard \`form\` views, often with a simplified layout, and are displayed in a modal dialog.]

**XML Structure:** 
[Placeholder: \`<form>\` views, often with \`<footer>\` for buttons like "Confirm", "Cancel", "Next", "Previous". Buttons usually call methods on the transient model.]
\`\`\`xml
<record id="view_my_wizard_form" model="ir.ui.view">
    <field name="name">my.wizard.form</field>
    <field name="model">my.wizard.model</field>
    <field name="arch" type="xml">
        <form string="My Wizard">
            <group>
                <field name="some_field"/>
                <field name="another_field"/>
            </group>
            <footer>
                <button name="action_confirm" string="Confirm" type="object" class="btn-primary"/>
                <button string="Cancel" class="btn-secondary" special="cancel"/>
            </footer>
        </form>
    </field>
</record>

<record id="action_my_wizard" model="ir.actions.act_window">
    <field name="name">Launch My Wizard</field>
    <field name="res_model">my.wizard.model</field>
    <field name="view_mode">form</field>
    <field name="target">new</field> <!-- 'new' opens in a dialog/modal -->
</record>
\`\`\`

**Python Backend:** 
[Placeholder: \`models.TransientModel\` defines fields to store wizard state and methods that are called by wizard buttons. These methods perform the wizard's logic.]

**Database Impact:** 
[Placeholder: Transient models are temporary and automatically cleared. The wizard's actions might create/update/delete persistent records or trigger other operations with database impact.]

## 3. Visual Documentation

**Mermaid Diagrams:**
\`\`\`mermaid
graph TD
    A[User Action Triggers Wizard] --> B(Wizard Form Appears - Step 1);
    B -- User Input --> C{Wizard Logic Processes Input};
    C -- More Steps --> D[Wizard Form - Step 2];
    C -- Final Step --> E[Action Performed / Data Processed];
    D --> C;
    E --> F[Wizard Closes / Result Shown];
\`\`\`
[Placeholder: Show the flow of a multi-step wizard or a simple input dialog.]

**Screenshots Description:** 
[Placeholder: Show an example of an Odoo wizard (modal dialog) with some input fields and action buttons in the footer (e.g., "Apply", "Cancel").]

**UI Layout:** 
[Placeholder: Modal dialog (popup) that overlays the current view. Contains a form view for inputs and a footer for action buttons.]

## 4. Functionality Description

**Core Features:** 
[Placeholder: Collect user input for a specific task. Guide users through sequential steps. Perform actions based on the collected input. Provide feedback to the user.]

**Configuration Options:** 
[Placeholder: Target (\`new\` for modal). \`special="cancel"\` for cancel buttons. Python methods define the logic of other buttons. Fields define the data to be collected.]

**Behavior Variations:** 
[Placeholder: Single-step or multi-step wizards. Can return another action (e.g., download a report, open another view) or simply close.]

**Integration Points:** 
[Placeholder: Launched from buttons in other views (form, tree), or from menu items. Can interact with any model or service in Odoo.]

## 5. Implementation Examples

**XML Configuration:**
\`\`\`xml
<!-- See XML example in Technical Details section above -->
<!-- Example of a button launching a wizard: -->
<button name="%(action_my_wizard)d" string="Open My Wizard" type="action"/>
\`\`\`

**Python Code:**
\`\`\`python
# Show related Python model/method implementations
from odoo import models, fields, api

class MyWizardModel(models.TransientModel):
    _name = 'my.wizard.model'
    _description = 'My Example Wizard'

    some_field = fields.Char(string="Some Input", required=True)
    another_field = fields.Integer(string="Another Input")
    # For multi-step wizards, a state field might be used
    # state = fields.Selection([('step1', 'Step 1'), ('step2', 'Step 2')], default='step1')

    def action_confirm(self):
        self.ensure_one()
        # Process the wizard data
        # Example: Create a record based on wizard input
        # self.env['res.partner'].create({'name': self.some_field, ...})
        # Or call a method on an active model if context is passed
        # active_ids = self.env.context.get('active_ids')
        # if active_ids:
        #     records = self.env[self.env.context.get('active_model')].browse(active_ids)
        #     records.some_method(self.some_field)
        
        # Can return an action, e.g., to close the wizard and refresh the view
        # or to download a file
        return {'type': 'ir.actions.act_window_close'}

    # For multi-step wizards:
    # def action_next_step(self):
    #     self.state = 'step2'
    #     return {
    #         'type': 'ir.actions.act_window',
    #         'res_model': self._name,
    #         'view_mode': 'form',
    #         'res_id': self.id,
    #         'target': 'new',
    #     }

\`\`\`

**JavaScript (if applicable):**
[Placeholder: Generally not required for standard wizard behavior unless highly custom client-side interactions are needed within the wizard's form view.]

## 6. Customization Guide

**Common Modifications:** 
[Placeholder: Adding new fields to the wizard. Changing the logic of action buttons. Adding new steps to a wizard. Modifying the view layout.]

**Extension Points:** 
[Placeholder: Inheriting the wizard's form view XML. Inheriting the \`models.TransientModel\` Python class to modify field definitions or method logic.]

**Best Practices:** 
[Placeholder: Keep wizards focused on a single task. Provide clear instructions and feedback to the user. Use \`models.TransientModel\` for temporary data. Ensure actions are idempotent if they might be retried.]

**Pitfalls to Avoid:** 
[Placeholder: Making wizards too complex or with too many steps. Storing permanent data directly in transient models. Poor error handling within wizard actions.]

## 7. Testing & Troubleshooting

**Test Scenarios:** 
[Placeholder: 
1. Launch the wizard from its intended entry point.
2. Test input validation for each field.
3. Test each action button (Confirm, Cancel, Next, Previous).
4. Verify the expected outcome after the wizard completes (e.g., record created/updated, report downloaded).
5. Test with different user roles if permissions affect wizard behavior.]

**Common Issues:** 
[Placeholder: Wizard not launching (check action definition, button XML). Errors during action execution (check Python code in the transient model). Data not saving/processing as expected.]

**Debug Tips:** 
[Placeholder: Add print statements or use a debugger in the Python methods of the transient model. Check server logs for errors. Inspect the \`context\` available to the wizard if it relies on active records or other context information.]
