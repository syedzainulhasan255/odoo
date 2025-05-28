# Action Buttons

## 1. Feature Overview

**Purpose:** 
Action Buttons in Odoo are interactive UI elements that allow users to trigger specific operations, workflows, or UI changes directly from various views (primarily Form, List, and Kanban). They serve as the primary mechanism for users to initiate actions such as saving records, transitioning a document through its lifecycle (e.g., confirming an order, posting an invoice), launching wizards for further input, running reports, or executing custom backend logic.

**Module Location:** 
The core functionality for rendering buttons is part of the \`web\` module. The definition of actions and the buttons themselves are in \`base\` (e.g., \`<button>\` tag in XML views) and extended by virtually every Odoo module that introduces specific business logic or workflows (e.g., \`account\` for invoice actions, \`sale\` for order confirmations).

**Dependencies:** 
Action buttons rely on:
*   Odoo's core view architecture (\`ir.ui.view\`) for their definition in XML.
*   The ORM for interacting with model data when \`type="object"\` buttons call model methods.
*   Action definitions (\`ir.actions.server\`, \`ir.actions.act_window\`) when \`type="action"\` buttons are used.
*   Model-level Python methods that execute the business logic.

**User Roles:** 
The visibility and availability of action buttons often depend on user permissions (Access Control Lists and Record Rules) for the specific action being triggered and the data record itself. For example, a "Confirm" button might only be visible to users in a certain group or when a record is in a specific state.

## 2. Technical Details

**Model/View Type:** 
Action Buttons are primarily defined within XML view definitions (\`ir.ui.view\`) using the \`<button>\` element. The behavior of these buttons is determined by their \`type\` attribute:
*   \`type="object"\`: Calls a Python method on the current model.
*   \`type="action"\`: Triggers an existing Odoo action, typically an \`ir.actions.act_window\` (to open a new view/wizard) or an \`ir.actions.server\` (to execute server-side code).
*   \`type="workflow"\` (Legacy): Used to signal a workflow engine, less common in modern Odoo versions.

**XML Structure:** 
Key attributes for the \`<button>\` element:
*   \`name\`:
    *   For \`type="object"\`: The name of the Python method to call on the model.
    *   For \`type="action"\`: The XML ID (e.g., \`%(action_xml_id)d\`) of the \`ir.actions.act_window\` or \`ir.actions.server\` record to execute.
*   \`string="Label"\`: The text displayed on the button.
*   \`type="object|action"\`: Defines the button's behavior.
*   \`class="oe_highlight|oe_link|btn-primary|btn-secondary|..."\`: CSS classes for styling. \`oe_highlight\` is commonly used for primary positive actions.
*   \`icon="fa-icon_name"\`: Specifies a FontAwesome icon to display on the button (e.g., \`icon="fa-check"\`).
*   \`attrs="{'invisible': [('field_name', 'operator', value)], 'readonly': [...]}"\`: Dynamically controls visibility and interactivity based on record field values.
*   \`confirm="Are you sure you want to proceed?"\`: Displays a confirmation dialog to the user before executing the button's action.
*   \`special="cancel"\`: A special attribute used in wizard footers to indicate a cancel button, which closes the wizard without further action.
*   \`states="state1,state2"\` (Legacy): A comma-separated list of states in which the button should be visible (often replaced by \`attrs\` in modern Odoo).

**Examples:**
\`\`\`xml
<!-- Object Button -->
<button name="action_post" string="Confirm" type="object" class="oe_highlight"
        attrs="{'invisible': [('state', '!=', 'draft')]}"/>

<!-- Action Button (launching a wizard/action) -->
<button name="%(account.action_account_invoice_payment_register)d" string="Register Payment"
        type="action" class="oe_highlight"
        attrs="{'invisible': ['|', ('state', '!=', 'posted'), ('payment_state', 'not in', ('not_paid', 'partial'))]}"/>

<!-- Button with confirmation -->
<button name="button_cancel" string="Cancel" type="object"
        confirm="Are you sure you want to cancel this entry? This will reset it to draft."
        attrs="{'invisible': [('state', '!=', 'posted')]}"/>

<!-- Special cancel button in a wizard footer -->
<button string="Cancel" class="btn-secondary" special="cancel"/>
\`\`\`

**Placement:**
Buttons are typically placed:
*   Inside the \`<header>\` element of a Form view for primary record actions.
*   Within the main \`<sheet>\` of a Form view for contextual actions related to specific field groups.
*   In the control panel associated with Tree or Kanban views (defined in the \`ir.actions.act_window\` XML).
*   Less commonly, directly within QWeb templates of Kanban cards or Tree view rows for record-specific actions.

**Python Backend:** 
*   **\`type="object"\` buttons:** Execute a Python method defined on the model associated with the current view. This method receives \`self\` (a recordset, often a single record if on a form view) and performs the business logic. Examples from \`account.move\`:
    *   \`action_post(self)\`: Changes the state of an invoice to 'posted'.
    *   \`button_cancel(self)\`: Changes the state of an invoice to 'cancel'.
    These methods often involve writing to fields (e.g., \`self.write({'state': 'new_state'})\`) and may return an action dictionary (e.g., to refresh the view or open another view) or \`True\`/\`None\`.
*   **\`type="action"\` buttons:** Execute a predefined Odoo Action record.
    *   \`ir.actions.act_window\`: Opens a new view (e.g., a wizard in a modal, a different list or form view).
    *   \`ir.actions.server\`: Executes server-side Python code, potentially performing complex operations, sending emails, or calling other model methods.

**Database Impact:** 
The database impact varies greatly depending on the button's action:
*   Opening a wizard or another view might have no direct database impact until the subsequent action is completed.
*   Calling an object method typically results in \`UPDATE\` statements (e.g., changing a record's state) or potentially \`INSERT\` or \`DELETE\` statements if the method creates/deletes related records.
*   Server actions can perform any range of database operations.

## 3. Visual Documentation

**Mermaid Diagrams:**
\`\`\`mermaid
graph TD
    A[User Clicks Button] --> B{Button Type?};
    B -- type="object" --> C[Python Method on Model Executed];
    B -- type="action" --> D[ir.actions.act_window or ir.actions.server Record Loaded];
    C --> F[Business Logic (e.g., self.write, state change)];
    D -- act_window --> G[New View/Wizard Displayed];
    D -- server_action --> H[Server Code Executed];
    F --> I[UI Update / Action Response];
    G --> I;
    H --> I;
\`\`\`

**Screenshots Description & UI Layout:** 
Action buttons are prominently featured in Odoo.
*   **Form View Headers:** The \`<header>\` section of a form view is the most common location for primary action buttons like "Save" (implicit), "Create" (implicit), "Confirm", "Post", "Cancel", "Set to Draft", etc. Status bars (\`widget="statusbar"\`) are also placed here to visually represent stages, and buttons often control transitions between these stages.
*   **List View Control Panel:** While not directly *in* the list view rows, actions that operate on selected records (e.g., "Delete", "Action" dropdown) are accessible from the control panel above the list when records are selected.
*   **Kanban View Cards:** Buttons can be embedded directly into Kanban card templates to perform record-specific actions (e.g., "Archive", "Quick Edit").

## 4. Functionality Description

**Core Features:** 
*   **Execute Model Methods:** Trigger Python logic defined directly on the record's model (\`type="object"\`).
*   **Launch Window Actions:** Open new views, forms, wizards, or reports (\`type="action"\` referencing \`ir.actions.act_window\`).
*   **Run Server Actions:** Execute predefined server-side code blocks (\`type="action"\` referencing \`ir.actions.server\`).
*   **Change Record State:** A common use case, often tied to a \`statusbar\` widget.
*   **Conditional Display/Interactivity:** Buttons can be dynamically shown, hidden, or made read-only using the \`attrs\` attribute based on the record's data.
*   **User Confirmation:** Prompt users for confirmation before executing potentially destructive or irreversible actions using the \`confirm\` attribute.

**Configuration Options:** 
*   \`name\`: Specifies the Python method or action XML ID.
*   \`string\`: The user-visible label of the button.
*   \`type\`: Determines if it calls a model method (\`object\`) or an Odoo action (\`action\`).
*   \`class\`: CSS classes for styling (e.g., \`oe_highlight\` for primary buttons, \`btn-secondary\` for secondary, \`oe_link\` for less prominent actions).
*   \`icon\`: A FontAwesome class to display an icon.
*   \`attrs\`: A dictionary to dynamically set attributes like \`invisible\`, \`readonly\` based on domain conditions. E.g., \`attrs="{'invisible': [('state', '=', 'posted')]}"\`.
*   \`confirm\`: A string for a confirmation message.
*   \`special="cancel"\`: Closes a wizard dialog without executing further actions.

**Behavior Variations:** 
*   **Highlighted Buttons:** Primary actions are often styled with \`oe_highlight\` or \`btn-primary\`.
*   **Conditional Logic:** \`attrs\` provide powerful dynamic control over a button's state.
*   **Server-Side vs. Client-Side:** Most Odoo buttons trigger server-side logic. Purely client-side button actions (without server interaction) require custom JavaScript, often by creating custom widgets or extending existing view controllers.

**Integration Points:** 
*   Action buttons are central to user interaction and workflow management in Odoo.
*   They bridge the UI (views) with backend business logic (Python model methods).
*   They are key components for launching wizards, generating reports, and navigating between different views or records.
*   They work in conjunction with the \`statusbar\` widget to visualize and control record progression through different states.

## 5. Implementation Examples

**XML Configuration (Inspired by \`account.move\`'s \`view_move_form\`):**
\`\`\`xml
<form string="Invoice">
    <header>
        <button name="action_post" string="Confirm" type="object" class="oe_highlight"
                attrs="{'invisible': [('state', '!=', 'draft')]}"/>
        <button name="button_cancel" string="Cancel" type="object"
                confirm="Are you sure you want to cancel this entry? This will reset it to draft."
                attrs="{'invisible': [('state', '!=', 'posted')]}"/>
        <button name="button_draft" string="Reset to Draft" type="object"
                attrs="{'invisible': [('state', 'not in', ('cancel', 'posted'))]}"/>
        <button name="%(account.action_account_invoice_payment_register)d" 
                id="account_invoice_payment_register_button"
                string="Register Payment" type="action" class="oe_highlight"
                attrs="{'invisible': ['|', ('state', '!=', 'posted'), ('payment_state', 'in', ('paid','in_payment'))]}"/>
        <field name="state" widget="statusbar" statusbar_visible="draft,posted"/>
    </header>
    <!-- ... other form elements ... -->
</form>
\`\`\`

**Python Code (Simplified methods from \`account.move\`):**
\`\`\`python
from odoo import models, fields, api, _
from odoo.exceptions import UserError

class AccountMoveButtonExamples(models.Model):
    _inherit = 'account.move' # In a real scenario, you'd inherit account.move

    # state = fields.Selection(selection=[
    #     ('draft', 'Draft'),
    #     ('posted', 'Posted'),
    #     ('cancel', 'Cancelled'),
    # ], string='Status', required=True, readonly=True, copy=False, default='draft')
    # payment_state = fields.Selection(selection=[
    #     ('not_paid', 'Not Paid'),
    #     ('in_payment', 'In Payment'),
    #     ('paid', 'Paid'),
    #     ('partial', 'Partially Paid'),
    # ], string="Payment Status", store=True, readonly=True, copy=False)


    def action_post(self):
        self.ensure_one()
        if self.state != 'draft':
            raise UserError(_("Only draft entries can be posted."))
        # Complex posting logic would be here...
        return self.write({'state': 'posted'})

    def button_cancel(self):
        self.ensure_one()
        if not self.journal_id.update_posted:
            raise UserError(_('You cannot cancel an entry in a journal that does not allow cancelling entries.'))
        # Complex cancellation logic...
        return self.write({'state': 'cancel'})

    def button_draft(self):
        self.ensure_one()
        # Complex logic to reset to draft...
        return self.write({'state': 'draft'})

    # No Python method needed for the "Register Payment" button as it's type="action"
    # and directly calls an ir.actions.act_window record.
\`\`\`

**JavaScript (if applicable):**
[Placeholder: Standard Odoo action buttons (\`type="object"\` or \`type="action"\`) are primarily driven by XML and Python. Custom JavaScript is generally reserved for:
*   Creating entirely new client-side actions/widgets that behave like buttons.
*   Modifying the behavior of existing buttons in very specific ways through patching view controllers (e.g., \`FormController\`, \`ListController\`). This is an advanced topic and should be approached with caution.]

## 6. Customization Guide

**Common Modifications:** 
*   Adding new buttons to forms or list view headers to trigger custom Python methods (\`type="object"\`).
*   Changing button labels (\`string\`), styling (\`class\`), or icons (\`icon\`).
*   Making buttons conditionally visible or readonly using \`attrs\` based on record state or user permissions.
*   Adding \`confirm="Are you sure?"\` messages to buttons that perform critical or irreversible actions.
*   Linking buttons to existing or new \`ir.actions.act_window\` (to open wizards, other views) or \`ir.actions.server\` (for more complex backend logic).

**Extension Points:** 
*   **XML View Inheritance:** The primary method. Use \`xpath\` to find existing elements (like \`<header>\`) and add or modify \`<button>\` elements within them.
*   **Model Method Definition (Python):** Add new Python methods to your models that can be called by \`type="object"\` buttons.
*   **Action Record Creation (XML/Python):** Define new \`ir.actions.act_window\` or \`ir.actions.server\` records if your button needs to trigger a standard action.
*   **JavaScript (Advanced):** For highly custom client-side interactions or button behaviors not achievable via XML/Python, you might need to extend JavaScript view controllers or create custom widgets.

**Best Practices:** 
*   Use \`oe_highlight\` (or \`btn-primary\`) for the main positive action(s) in a view.
*   Use \`attrs\` extensively for conditional visibility/read-only states to keep the UI clean and context-aware.
*   Keep Python methods called by buttons focused on a single, clear action. If multiple steps or complex input is needed, consider using a wizard launched by the button.
*   Provide clear and concise button labels.
*   Always add \`confirm\` messages for actions that are destructive or not easily undone.

**Pitfalls to Avoid:** 
*   Overcrowding views (especially form headers) with too many buttons. Consider grouping related actions under a dropdown if necessary.
*   Inconsistent button placement or styling across different views/modules.
*   Buttons that perform long-running operations without providing feedback to the user (can make the UI feel unresponsive).
*   Modifying core button actions directly; prefer inheriting views or models to make changes.

## 7. Testing & Troubleshooting

**Test Scenarios:** 
*   **Visibility & Read-only State:** Verify buttons appear/disappear or become active/inactive correctly based on different record states and user roles (test \`attrs\` thoroughly).
*   **Action Execution:**
    *   For \`type="object"\` buttons: Ensure the correct Python method is called and executes the intended logic (e.g., state changes, data updates).
    *   For \`type="action"\` buttons: Ensure the correct window action (wizard, new view) or server action is triggered.
*   **Confirmation Dialogs:** Test that \`confirm\` messages appear when expected and that "OK" and "Cancel" options work correctly.
*   **User Permissions:** Test button behavior with users who have and do not have the necessary permissions for the action.
*   **Contextual Behavior:** If a button's behavior depends on the context (e.g., selected records in a list view), test these scenarios.

**Common Issues:** 
*   **Button not appearing:** Check \`attrs\` conditions, group memberships required for visibility (if any in \`groups\` attribute or Python code), and ensure the XML is correctly structured.
*   **Button doing nothing or wrong action:**
    *   For \`type="object"\`: Verify the \`name\` attribute matches an existing method in the model. Check for Python errors in the method's code.
    *   For \`type="action"\`: Ensure the XML ID in the \`name\` attribute (e.g., \`%(my_module.my_action_id)d\`) is correct and points to a valid \`ir.actions.act_window\` or \`ir.actions.server\` record.
*   **Permissions errors:** The user might lack permissions for the Python method or the target action/model.
*   **Incorrect \`attrs\` domain:** Syntax errors in the \`attrs\` domain can cause unexpected behavior.

**Debug Tips:** 
*   **Developer Mode:**
    *   "Edit View: Form/List/Kanban" to inspect button XML attributes (\`name\`, \`type\`, \`attrs\`, \`string\`, \`class\`, \`icon\`).
    *   For \`type="action"\` buttons, "View Metadata" (from the action that uses the view) or searching for the action record directly can help verify its definition.
*   **Browser Developer Tools:** Useful for client-side issues, although most button logic is server-side. Can help inspect \`attrs\` if they are not behaving as expected.
*   **Python Debugger (\`pdb\` or IDE debugger):** Set breakpoints in Python methods called by \`type="object"\` buttons or in server action code to step through the logic.
*   **Odoo Server Logs:** Check for any tracebacks or error messages when a button is clicked, especially for \`type="object"\` methods or server actions.
*   **Logging:** Add \`_logger.info(...)\` or \`print()\` statements in Python methods to trace execution and variable values.
