# Wizard Forms

## 1. Feature Overview

**Purpose:** 
Odoo Wizard Forms are specialized dialogs designed to guide users through multi-step processes, collect specific input for operations, or configure actions. They typically use temporary data models (\`models.TransientModel\`) that are not meant for permanent storage but rather to hold data for the duration of the wizard's interaction. Wizards are often employed for complex operations like generating reports with specific parameters, configuring settings, performing batch updates with user-defined options, or handling intermediate steps in a larger workflow.

**Module Location:** 
The core framework for wizards (transient models, modal dialogs) is provided by the \`base\` module. Specific wizards are then defined within the modules where their functionality is relevant (e.g., \`account\` for wizards related to payment registration or invoice reversal, \`hr\` for employee departure wizards).

**Dependencies:** 
Wizards primarily depend on:
*   \`models.TransientModel\`: The base class for wizard models.
*   \`ir.ui.view\` (type \`form\`): For defining the wizard's user interface.
*   \`ir.actions.act_window\`: To launch the wizard as a modal dialog.

**User Roles:** 
Access to a wizard and its functionality depends on the user's permissions for the action that launches the wizard and any underlying models or operations the wizard interacts with.

## 2. Technical Details

**Model/View Type:** 
*   **Model:** Wizards are typically backed by models inheriting from \`models.TransientModel\`. Records of transient models are temporary and are automatically cleared from the database periodically by a system cron job. They are not intended for storing persistent business data.
*   **View:** The user interface of a wizard is a standard Odoo \`form\` view, but it is displayed within a modal dialog (popup window) that overlays the current screen, temporarily disabling interaction with the background view.

**XML Structure:** 
Defining a wizard involves two main XML parts: the action to launch it and the form view for its UI.

1.  **Form View Definition (\`ir.ui.view\`):**
    \`\`\`xml
    <record id="view_example_wizard_form" model="ir.ui.view">
        <field name="name">example.wizard.form</field>
        <field name="model">example.wizard</field> <!-- Name of the TransientModel -->
        <field name="arch" type="xml">
            <form string="Example Wizard Title">
                <group>
                    <field name="field_one"/>
                    <field name="field_two" options="{'no_create_edit': True}"/>
                </group>
                <group string="More Options">
                    <field name="another_option_field"/>
                </group>
                <footer>
                    <button name="action_perform_operation" string="Confirm Action" type="object" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>
    \`\`\`

2.  **Action Window (\`ir.actions.act_window\`) to Launch the Wizard:**
    \`\`\`xml
    <record id="action_launch_example_wizard" model="ir.actions.act_window">
        <field name="name">Launch Example Wizard</field>
        <field name="res_model">example.wizard</field> <!-- Name of the TransientModel -->
        <field name="view_mode">form</field>
        <field name="target">new</field> <!-- 'new' is crucial for opening in a modal dialog -->
        <!-- Optional: Bind the wizard to a model, making it appear in the "Action" menu -->
        <!-- <field name="binding_model_id" ref="model_your_source_model"/> -->
        <!-- <field name="binding_view_types">form,list</field> -->
    </record>
    \`\`\`

**Python Backend:** 
*   The wizard's logic is implemented in a Python class inheriting from \`models.TransientModel\`.
*   Fields defined on this transient model hold the data entered by the user or used to control the wizard's flow.
*   Methods on the transient model are called when users click buttons (typically \`type="object"\`) in the wizard's form view. These methods contain the business logic to perform the desired operations.
*   Wizards often operate on the context of the view from which they were launched. They can access \`active_id\` (for a single record context, e.g., from a form view) or \`active_ids\` (for multiple records, e.g., selected from a list view) from \`self.env.context\`. The \`active_model\` is also available.
    \`\`\`python
    # Example: Accessing active_ids in a wizard method
    active_ids = self.env.context.get('active_ids', [])
    records = self.env[self.env.context.get('active_model')].browse(active_ids)
    \`\`\`

**Database Impact:** 
*   Records of \`models.TransientModel\` are stored in the database but are meant to be short-lived. A system cron job (\`base.ir_autovacuum\`) periodically deletes old transient records.
*   The primary database impact of a wizard comes from the actions its methods perform on *persistent* models (e.g., creating an \`account.move\` record, updating \`res.partner\` records, etc.).

## 3. Visual Documentation

**Mermaid Diagrams:**
\`\`\`mermaid
graph TD
    A[User Action Triggers Wizard (e.g., Button Click)] --> B[ir.actions.act_window (target='new')];
    B --> C[Wizard Form (TransientModel) Renders in Modal];
    C -- User Enters Data --> D(Wizard Fields Hold Data);
    D --> E{User Clicks Action Button (e.g., "Confirm")};
    E -- type="object" --> F[Python Method on TransientModel Executed];
    F -- Uses self.env.context (active_ids, etc.) --> G[Performs Logic (e.g., CRUD on persistent models)];
    G --> H[Optional: Returns another action (report, view refresh)];
    H --> I[Wizard Closes];
    E -- special="cancel" --> I;
    C -- Timeout/External Click (if configured) --> I;
\`\`\`

**Screenshots Description & UI Layout:** 
An Odoo wizard appears as a modal dialog (a pop-up window) that overlays the current screen, temporarily disabling interaction with the background view. The content of the wizard is a standard Odoo form view, typically simplified to include only necessary fields for the specific operation. It usually has:
*   A **title bar** displaying the wizard's name (from the \`string\` attribute of the \`<form>\` tag or the action).
*   A **main content area** with fields (e.g., text inputs, selection dropdowns, checkboxes) organized using \`<group>\` elements.
*   A **footer** (\`<footer>\`) containing action buttons like "Confirm", "Apply", "Generate", "Continue", or "Cancel". Primary action buttons are often highlighted.

## 4. Functionality Description

**Core Features:** 
*   **User Input Collection:** Gather specific parameters or data from the user required for an operation.
*   **Multi-Step Guidance:** Can be designed to guide users through a sequence of steps by returning actions that reopen the wizard (potentially with a different view or state).
*   **Task Execution:** Perform backend operations based on the collected input (e.g., creating records, updating data, generating reports, sending emails).
*   **User Feedback:** Can display messages, warnings, or results to the user, or redirect to other views.

**Configuration Options:** 
*   **\`target="new"\`** in \`ir.actions.act_window\`: This is essential to make the form view open as a modal wizard.
*   **\`special="cancel"\`** on a \`<button>\` in the \`<footer>\`: Provides a standard way to close the wizard without performing the primary action.
*   Python methods linked to \`type="object"\` buttons define the wizard's core logic.
*   Fields on the \`models.TransientModel\` define the data the wizard collects and processes.

**Behavior Variations:** 
*   **Single-step wizards:** Collect all input in one form and perform an action.
*   **Multi-step wizards:** Guide the user through several forms, often by managing a \`state\` field on the transient model and returning an action to reopen the wizard on the same record.
*   Can return another action upon completion (e.g., download a report, open a list view of created records) or simply close.

**Integration Points:** 
*   Wizards are typically launched from:
    *   Buttons in Form or Tree views (e.g., "Reverse Journal Entry" button on an account move form).
    *   Menu items (e.g., "Settings" wizards).
    *   Server actions.
*   They can interact with any Odoo model or service to perform their defined tasks.
*   Often use the context (\`active_id\`, \`active_ids\`, \`active_model\`) passed from the view that launched them to operate on specific records.

## 5. Implementation Examples

**XML Configuration (Inspired by \`account.move.reversal\`):**

*Action to launch the wizard:*
\`\`\`xml
<record id="action_account_move_reversal_example" model="ir.actions.act_window">
    <field name="name">Reverse Journal Entry Example</field>
    <field name="res_model">account.move.reversal.example</field> <!-- Name of our TransientModel -->
    <field name="view_mode">form</field>
    <field name="target">new</field>
    <field name="binding_model_id" ref="account.model_account_move"/> <!-- To appear in "Action" menu of account.move -->
    <field name="binding_view_types">form,list</field>
</record>
\`\`\`

*Form view for the wizard:*
\`\`\`xml
<record id="view_account_move_reversal_form_example" model="ir.ui.view">
    <field name="name">account.move.reversal.form.example</field>
    <field name="model">account.move.reversal.example</field>
    <field name="arch" type="xml">
        <form string="Reverse Journal Entry">
            <group>
                <field name="reason" string="Reason (Optional)"/>
                <field name="date_mode"/>
                <field name="date" attrs="{'invisible': [('date_mode', '=', 'entry')]}"/>
                <field name="journal_id" options="{'no_create': True}"/>
            </group>
            <footer>
                <button string="Reverse" name="action_reverse_moves" type="object" class="btn-primary" data-hotkey="q"/>
                <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
            </footer>
        </form>
    </field>
</record>
\`\`\`

**Python Code (Simplified \`AccountMoveReversal\`):**
\`\`\`python
from odoo import models, fields, api, _
from odoo.exceptions import UserError

class AccountMoveReversalExample(models.TransientModel):
    _name = 'account.move.reversal.example'
    _description = 'Credit Note / Reversal Wizard Example'

    reason = fields.Char(string='Reason')
    date_mode = fields.Selection(
        selection=[('entry', 'Use Journal Entry Date'), ('custom', 'Custom Date')],
        string='Reversal Date', default='custom', required=True)
    date = fields.Date(string='Custom Date', default=fields.Date.context_today)
    journal_id = fields.Many2one(
        'account.journal', string='Use Specific Journal',
        help='If empty, uses the journal of the journal entry to reverse.')

    @api.model
    def default_get(self, fields_list):
        res = super().default_get(fields_list)
        # Example: Pre-fill journal if only one suitable journal exists for active model
        active_model = self.env.context.get('active_model')
        active_ids = self.env.context.get('active_ids')
        if active_model == 'account.move' and active_ids:
            # move = self.env['account.move'].browse(active_ids[:1])
            # res['journal_id'] = move.journal_id.id # Example prefill
            pass
        return res

    def action_reverse_moves(self):
        self.ensure_one()
        active_ids = self.env.context.get('active_ids', [])
        if not active_ids:
            raise UserError(_("No journal entries selected to reverse."))
        
        moves_to_reverse = self.env['account.move'].browse(active_ids)
        
        # Simplified logic:
        reversed_move_ids = []
        for move in moves_to_reverse:
            # In real scenario, call the actual _reverse_moves method
            # new_move = move._reverse_moves([{'date': self.date, 'journal_id': self.journal_id.id}], cancel=False)
            # reversed_move_ids.extend(new_move.ids)
            self.env['account.move'].create({
                'name': f"REV/{move.name or ''}",
                'journal_id': self.journal_id.id or move.journal_id.id,
                'date': self.date if self.date_mode == 'custom' else move.date,
                'ref': _("Reversal of: %s, %s") % (move.name, self.reason or ''),
                'state': 'draft',
                # ... copy lines reversed ...
            })
            print(f"Simulating reversal for move ID {move.id} with reason: {self.reason}")

        # Could return an action to view the newly created reversed moves
        return {'type': 'ir.actions.act_window_close'}
\`\`\`

**JavaScript (if applicable):**
[Placeholder: Generally not required for standard wizard behavior. Custom JavaScript might be used for highly dynamic client-side interactions within the wizard's form view, such as complex conditional logic for field visibility or custom widget behaviors that go beyond standard Odoo capabilities.]

## 6. Customization Guide

**Common Modifications:** 
*   Adding new fields to the wizard's transient model to collect more data.
*   Changing the logic of the action buttons (Python methods) in the transient model.
*   Adding new steps to a wizard (often by managing a \`state\` field and returning actions to reopen the wizard).
*   Modifying the form view layout (XML) of the wizard for better usability or to include new fields.
*   Changing button strings, classes, or adding \`confirm\` messages.

**Extension Points:** 
*   **XML View Inheritance:** Inherit the wizard's form view (\`ir.ui.view\`) using \`xpath\` to add or modify fields, groups, buttons, or attributes.
*   **Transient Model Inheritance (Python):** Inherit the wizard's \`models.TransientModel\` class to add new fields, override existing methods (e.g., \`action_confirm\`), or add new methods for new buttons.
*   **Action Window Modification:** Modify the \`ir.actions.act_window\` that launches the wizard, for example, to change its \`name\`, \`binding_model_id\`, or \`context\`.

**Best Practices:** 
*   Keep wizards focused on a single, well-defined task or workflow.
*   Provide clear instructions and feedback to the user within the wizard (e.g., using \`help\` attributes on fields, clear button labels).
*   Use \`models.TransientModel\` for data that is only needed for the duration of the wizard.
*   Ensure actions performed by the wizard are idempotent if they might be retried by the user (e.g., due to network issues or accidental double-clicks).
*   Pass necessary context (like \`active_ids\`) from the launching action to the wizard.

**Pitfalls to Avoid:** 
*   Creating overly complex wizards with too many steps or fields, which can confuse users.
*   Attempting to store permanent business data directly in transient models instead of creating/updating persistent records.
*   Poor error handling within the wizard's action methods, leading to unhelpful error messages for the user.
*   Not clearly indicating the outcome of the wizard's operation.

## 7. Testing & Troubleshooting

**Test Scenarios:** 
*   Launch the wizard from all intended entry points (buttons, menus).
*   Test input validation for each field in the wizard (required fields, correct data types).
*   Test each action button (e.g., "Confirm", "Apply", "Cancel", "Next", "Previous" for multi-step).
*   Verify the expected outcome after the wizard completes its primary action (e.g., records created/updated, report downloaded, state changed).
*   If the wizard is multi-step, test navigation between steps.
*   Test with different user roles if permissions might affect wizard behavior or data access.
*   Test with edge cases or invalid input to ensure graceful error handling.
*   Verify that \`active_id\` or \`active_ids\` from the context are correctly used if the wizard operates on specific records.

**Common Issues:** 
*   **Wizard not launching:** Check the \`name\` in the button XML matches the action's XML ID or method name. Ensure the \`res_model\` in the action matches the wizard's model name. Verify \`target="new"\`.
*   **Errors during action execution:** Debug the Python method called by the button in the transient model. Check for logic errors, incorrect field access, or issues with data passed from the wizard form.
*   **Data not saving/processing as expected:** Ensure the wizard's action method correctly reads field values from \`self\` (the wizard record) and performs the intended ORM operations on persistent models.
*   **Context issues:** If the wizard relies on \`active_id\` or \`active_ids\`, ensure this context is correctly passed from the action that launches the wizard.
*   **Transient model data lost unexpectedly:** Remember transient records are temporary. If data needs to persist beyond the wizard's immediate execution, it must be saved to a regular (persistent) model.

**Debug Tips:** 
*   **Python Debugger:** Place breakpoints (\`pdb.set_trace()\`) in the Python methods of your \`models.TransientModel\` to inspect field values (\`self.field_name\`), context (\`self.env.context\`), and execution flow.
*   **Odoo Server Logs:** Check the server logs for any tracebacks or error messages that occur when the wizard is launched or its actions are executed.
*   **Developer Mode:**
    *   "Edit Action" (if launched from a button linked to an action) to inspect the \`ir.actions.act_window\` definition.
    *   "Edit View: Form" (when the wizard is open) to inspect its XML structure.
*   **Logging:** Add \`_logger.info(...)\` or \`print()\` statements in your wizard's Python methods to output variable values or trace execution paths to the server log.
*   **Inspect Context:** In Python methods, print or log \`self.env.context\` to understand what data (like \`active_ids\`) is available to your wizard.
