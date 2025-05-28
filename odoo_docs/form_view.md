# Form View

## 1. Feature Overview

**Purpose:** 
Odoo Form Views provide a comprehensive and user-friendly interface for displaying, creating, and editing a single record of a specific model. They are the primary means for detailed data entry and review, organizing fields logically and offering various interactive elements like buttons, status bars, and integrated communication tools (chatter).

**Module Location:** 
Core form view rendering is handled by the \`web\` module. The basic structure and model definitions rely on the \`base\` module. Specific applications (e.g., \`account\`, \`sale\`) then define their own form views for their models.

**Dependencies:** 
Form views fundamentally rely on:
*   The Odoo ORM (Object-Relational Mapper) for data interaction.
*   The core view architecture (\`ir.ui.view\`).
*   View inheritance mechanisms for customization.
*   Basic field types defined in Odoo models.

**User Roles:** 
Access to form views and the ability to perform operations (create, read, write, delete) are primarily determined by the user's access rights (ACLs and record rules) defined for the underlying data model.

## 2. Technical Details

**Model/View Type:** 
Form views are technically defined as records of the \`ir.ui.view\` model with the \`type\` attribute set to \`form\`.

**XML Structure:** 
The structure of a form view is defined using XML. Key elements include:
*   \`<form>\`: The root element. Common attributes:
    *   \`string\`: The title of the form (often overridden by \`display_name\` of the record).
    *   \`edit="true/false"\`: Allows/disallows editing of existing records.
    *   \`create="true/false"\`: Allows/disallows creation of new records through this view.
    *   \`delete="true/false"\`: Allows/disallows deletion of records from this view.
*   \`<header>\`: Contains action buttons and statusbar widgets.
*   \`<sheet>\`: The main content area of the form, typically with a responsive layout.
*   \`<group>\`: Used to group fields, often displayed as a two-column layout. Can have a \`string\` attribute for a group title.
*   \`<notebook>\`: Allows for tabbed content sections.
    *   \`<page string="Page Title">\`: Defines a single tab within a notebook.
*   \`<field name="field_name">\`: Displays a field from the model. Common attributes:
    *   \`widget="widget_name"\`: Specifies a custom widget for rendering (e.g., \`many2many_tags\`, \`monetary\`).
    *   \`options="{'option_key': 'value'}"\`: Passes options to widgets.
    *   \`invisible="[(condition)]"\`: Dynamically hides the field.
    *   \`readonly="[(condition)]"\`: Dynamically makes the field read-only.
    *   \`required="[(condition)]"\`: Dynamically makes the field mandatory.
    *   \`nolabel="1"\`: Hides the field's label.
*   \`<button>\`: Defines an action button. Common attributes:
    *   \`name="method_or_action_name"\`: The Python method name (for \`type="object"\`) or action ID (for \`type="action"\`).
    *   \`string="Label"\`: The button's text.
    *   \`type="object/action"\`: Specifies the button's behavior.
    *   \`class="oe_highlight/oe_link"\`: CSS classes for styling.
    *   \`icon="fa-icon_name"\`: Displays an icon on the button.
    *   \`attrs="{'invisible': [('field_name', 'operator', value)]}"\`: Dynamically hides/shows the button.
    *   \`confirm="Confirmation message"\`: Shows a confirmation dialog before executing.

**Python Backend:** 
*   Form views are primarily backed by models inheriting from \`models.Model\` (for persistent data) or \`models.TransientModel\` (for wizards/temporary data).
*   **Key ORM methods** implicitly used by form views include:
    *   \`default_get()\` : Provides default values when creating a new record.
    *   \`create()\` : Called when saving a new record.
    *   \`read()\` : Called to fetch and display record data.
    *   \`write()\` : Called when saving changes to an existing record.
    *   \`unlink()\` : Called when deleting a record.
*   \`@api.onchange('field_name')\` methods are crucial for dynamic behavior. They are triggered when the value of a specified field changes in the form, allowing other fields to be updated, domains to be changed, or warnings to be displayed. For example, in \`account.move\`, the \`_onchange_partner_id\` method updates fields like \`fiscal_position_id\` based on the selected partner.
*   **Compute methods** (fields defined with \`compute='_compute_method_name'\`) are used to calculate field values dynamically. These values are then displayed in the form view. For instance, \`_compute_amount\` in \`account.move\` calculates various totals.

**Database Impact:** 
Form view operations directly translate to CRUD (Create, Read, Update, Delete) operations on the underlying model's database table.
*   Saving a new record inserts a row.
*   Editing a record updates a row.
*   Deleting a record removes a row.
*   Loading a form reads data from one or more rows.

## 3. Visual Documentation

**Mermaid Diagrams:**
\`\`\`mermaid
graph TD
    subgraph Data Loading
        A[Open Form View] --> B(Call default_get or read);
        B --> C[Model Data Fetched];
        C --> D[Form Renders with Data & Widgets];
    end
    subgraph User Interaction
        D --> E{User Edits Field};
        E -- Onchange Field --> F(@api.onchange Triggered);
        F --> G[Field Values Updated Dynamically];
        G --> D;
        E -- Other Field --> D;
        D --> H[User Clicks Button];
    end
    subgraph Action Execution
        H -- Save --> I(Call create/write);
        H -- Action/Object Button --> J[Execute Python Method / Window Action];
        I --> K[Database CRUD];
        J --> K;
        K --> L[View Refreshes / Action Response];
    end
\`\`\`

**Screenshots Description:** 
A typical Odoo form view presents a structured layout for a single record. At the top, a **header** area usually contains action buttons (e.g., "Save", "Confirm", "Create Invoice") and a status bar (\`widget="statusbar"\`) indicating the record's stage in a workflow. The main content is within a **sheet**, often organized into logical sections using \`<group>\` elements (typically creating a two-column field layout) and \`<notebook>\` with \`<page>\` elements for tabbed information. Fields are displayed with labels and appropriate input widgets. At the bottom, the **chatter** section provides a history of changes, internal notes, and messaging capabilities.

**UI Layout:** 
The general layout is designed for clarity and ease of data entry. The \`<header>\` provides primary actions. The \`<sheet>\` contains the bulk of the data fields, often using responsive \`<group>\` elements for field organization. \`<notebook>\` and \`<page>\` tags are used to manage larger amounts of information by splitting it into tabs. The right side often features the chatter component for communication and audit trail.

## 4. Functionality Description

**Core Features:** 
*   Display detailed information of a single record.
*   Allow creation and editing of record fields using various input widgets tailored to data types (e.g., \`Char\`, \`Integer\`, \`Selection\`, \`Many2one\`, \`Date\`, \`Datetime\`, \`Binary\`, \`Html\`).
*   Trigger model-specific actions or workflows via buttons.
*   Display workflow status using \`statusbar\` widgets.
*   Integrate communication tools and record history via the chatter (\`mail.thread\` integration).
*   Support for conditional visibility and read-only states for fields and buttons.

**Configuration Options:** 
*   \`attrs\`: Allows dynamic control over field/button attributes (e.g., \`invisible\`, \`readonly\`, \`required\`) based on other field values or conditions. Example: \`attrs="{'invisible': [('state', '=', 'draft')]}"\`.
*   Field-specific \`widget\` attribute to use specialized UI elements (e.g., \`widget="monetary"\`, \`widget="priority"\`, \`widget="many2many_tags"\`).
*   Field-specific \`options\` attribute to pass parameters to JavaScript widgets, e.g., \`options="{'currency_field': 'my_currency_field_name'}"\`.
*   Form view attributes like \`edit\`, \`create\`, \`delete\` to control record operations.

**Behavior Variations:** 
*   **Read-only mode:** All fields are non-editable, typically when a record is in a certain state or due to user permissions.
*   **Edit mode:** Fields become editable for data modification.
*   \`invisible\` attribute: Fields or elements can be completely hidden based on conditions.
*   \`readonly\` attribute: Fields can be displayed but not edited based on conditions.
*   Behavior of specific widgets (e.g., a \`Many2one\` field might have options to create or search more).

**Integration Points:** 
*   Form views are launched from various places:
    *   Clicking a record in a Tree or Kanban view.
    *   Menu items (often opening a new form or a specific record).
    *   Buttons in other views or wizards.
*   Buttons within a form view can:
    *   Execute Python methods on the current model (\`type="object"\`).
    *   Trigger other Odoo actions like opening a wizard, another view, or a report (\`type="action"\`).
*   Integrates with the \`mail\` module for chatter functionality (tracking field changes, logging notes, sending messages).

## 5. Implementation Examples

**XML Configuration:**

*Simple Partner Form (existing example):*
\`\`\`xml
<record id="view_partner_form_example" model="ir.ui.view">
    <field name="name">res.partner.form.example</field>
    <field name="model">res.partner</field>
    <field name="arch" type="xml">
        <form string="Partner">
            <sheet>
                <group>
                    <group>
                        <field name="name"/>
                        <field name="phone"/>
                        <field name="email"/>
                    </group>
                    <group>
                        <field name="street"/>
                        <field name="city"/>
                        <field name="country_id"/>
                    </group>
                </group>
            </sheet>
        </form>
    </field>
</record>
\`\`\`

*More Complex Example (inspired by \`account.move\`'s \`view_move_form\`):*
\`\`\`xml
<record id="view_invoice_form_example" model="ir.ui.view">
    <field name="name">account.invoice.form.example</field>
    <field name="model">account.move</field> <!-- Using account.move as an example model -->
    <field name="arch" type="xml">
        <form string="Invoice" class="o_form_readonly_mode_black_border" edit="true" create="true" delete="true">
            <header>
                <button name="action_post" string="Confirm" class="oe_highlight" type="object"
                        attrs="{'invisible': [('state', '!=', 'draft')]}"/>
                <button name="%(account.action_account_invoice_payment_register)d" string="Register Payment"
                        type="action" class="oe_highlight"
                        attrs="{'invisible': ['|', ('state', '!=', 'posted'), ('payment_state', 'not in', ('not_paid', 'partial'))]}"/>
                <field name="state" widget="statusbar" statusbar_visible="draft,posted"/>
            </header>
            <sheet>
                <div class="oe_title">
                    <h1>
                        <field name="name" readonly="1"/>
                    </h1>
                </div>
                <group>
                    <group id="header_left_group">
                        <field name="partner_id" widget="res_partner_many2one"
                               options="{'no_quick_create': True}"/>
                        <field name="invoice_date"/>
                    </group>
                    <group id="header_right_group">
                        <field name="journal_id" options="{'no_create': True, 'no_edit': True}"/>
                        <field name="company_id" groups="base.group_multi_company" readonly="1"/>
                        <field name="currency_id" groups="base.group_multi_currency"/>
                    </group>
                </group>
                <notebook>
                    <page id="invoice_tab" string="Invoice Lines">
                        <field name="invoice_line_ids">
                            <tree editable="bottom">
                                <field name="product_id"/>
                                <field name="quantity"/>
                                <field name="price_unit"/>
                                <field name="price_subtotal" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            </tree>
                        </field>
                    </page>
                    <page id="other_info_tab" string="Other Info">
                        <group>
                            <field name="narration" placeholder="Internal notes..."/>
                        </group>
                    </page>
                </notebook>
                 <field name="amount_total" widget="monetary" options="{'currency_field': 'currency_id'}"/>
            </sheet>
            <div class="oe_chatter">
                <field name="message_follower_ids"/>
                <field name="activity_ids"/>
                <field name="message_ids"/>
            </div>
        </form>
    </field>
</record>
\`\`\`

**Python Code:**

*Simple Partner Example (existing):*
\`\`\`python
from odoo import models, fields

class ExamplePartner(models.Model):
    _inherit = 'res.partner'
    example_custom_field = fields.Char(string="Custom Field for Form View")

    def custom_button_action(self):
        self.ensure_one()
        return {'type': 'ir.actions.act_window_close'}
\`\`\`

*More Complex Examples (inspired by \`account.move\`):*
\`\`\`python
from odoo import models, fields, api, _
from odoo.exceptions import UserError

class AccountMoveFormExample(models.Model):
    _inherit = 'account.move' # Example, actual model is account.move

    # Simplified action method
    def action_post(self):
        self.ensure_one()
        if self.state != 'draft':
            raise UserError(_("Only draft invoices can be posted."))
        self.write({'state': 'posted'})
        return True

    # Simplified onchange method
    @api.onchange('partner_id')
    def _onchange_partner_id_example(self):
        if self.partner_id:
            # Example: Set a default journal based on partner type or other criteria
            # This is a simplified example; real logic can be more complex
            if self.partner_id.is_company:
                # self.journal_id = self.env['account.journal'].search([('type', '=', 'sale')], limit=1)
                pass # Placeholder for actual logic
        else:
            # self.journal_id = False
            pass # Placeholder


    # Simplified compute method
    # amount_total = fields.Monetary(compute='_compute_amount_example', store=True)
    # @api.depends('invoice_line_ids.price_subtotal')
    # def _compute_amount_example(self):
    #     for move in self:
    #         total = 0
    #         for line in move.invoice_line_ids:
    #             total += line.price_subtotal
    #         move.amount_total = total
    pass # Placeholder as actual fields are complex
\`\`\`

**JavaScript (if applicable):**
[Placeholder: JavaScript is primarily used for custom field widgets, client-side validations, or complex dynamic interactions within the form view that go beyond what \`attrs\` and \`@api.onchange\` can achieve. Standard form behavior is handled by the Odoo web client's JavaScript framework (Owl).]

## 6. Customization Guide

**Common Modifications:** 
*   Adding or removing fields from the view.
*   Changing field labels (\`string\` attribute).
*   Making fields conditionally visible, readonly, or required using \`attrs\`.
*   Adding new buttons to trigger custom Python methods or existing actions.
*   Modifying button actions, labels, or conditional visibility.
*   Rearranging fields and groups for better usability.
*   Applying existing widgets to fields or configuring widget options.

**Extension Points:** 
*   **XML View Inheritance:** Use \`xpath\` expressions to target elements in the base form view and modify them (add, remove, change attributes). This is the most common way to customize forms.
*   **Model Method Overrides (Python):** Override existing Python methods (e.g., \`create\`, \`write\`, \`default_get\`, or custom action methods) to alter backend logic associated with the form.
*   **New Model Methods (Python):** Add new methods to the model that can be called by new buttons (\`type="object"\`).
*   **JavaScript Widgets (Owl):** Develop custom JavaScript widgets for fields requiring unique UI interactions not covered by standard widgets. Register these in the \`fields\` registry.

**Best Practices:** 
*   Use \`xpath\` expressions that are as specific as possible to minimize conflicts with other modules or future Odoo updates.
*   Keep Python logic within model methods rather than embedding complex logic directly in \`ir.actions.server\` if possible.
*   Utilize \`attrs\` for dynamic UI changes based on record state or field values, as this is processed client-side and is efficient.
*   Group related fields together logically using \`<group>\` and \`<page>\` elements.

**Pitfalls to Avoid:** 
*   Over-modifying complex core views (like \`account.move.form\`) can lead to significant maintenance overhead during Odoo upgrades.
*   Introducing overly complex domains in \`attrs\`, which can be hard to debug and might impact performance.
*   Breaking core business logic by incorrectly overriding model methods.
*   Not testing customizations thoroughly across different user roles and record states.

## 7. Testing & Troubleshooting

**Test Scenarios:** 
*   **Create Mode:** Verify default values, required field enforcement, correct data saving.
*   **Edit Mode:** Verify data loading, field modifications, saving changes, conditional logic (\`attrs\`).
*   **Read-only Mode:** Ensure fields are not editable when they should be.
*   **Button Actions:** Test each button for correct functionality (method execution, wizard opening, state changes, confirmation messages).
*   **Conditional Logic:** Thoroughly test \`attrs\` for visibility, readonly, and required states with different data combinations.
*   **Required Fields:** Ensure validation messages appear and prevent saving if required fields are empty.
*   **Onchange Behavior:** Test that fields update as expected when related fields are changed.
*   **User Permissions:** Test form accessibility and button/field behavior with different user roles.

**Common Issues:** 
*   **Fields not appearing:** Check XML for typos, \`invisible\` attributes, field definition in the model, and user access rights to the field.
*   **Buttons not working/appearing:** Check method name or action ID in XML, Python method implementation, \`attrs\` conditions, and user permissions for the action/method.
*   **\`attrs\` not evaluating correctly:** Double-check domain syntax and field names used in \`attrs\`.
*   **Data not saving:** Look for server-side errors, constraint violations, or issues in \`create\`/\`write\` methods.
*   **Performance:** Complex computed fields or onchanges, or too many fields on a form can slow down loading.

**Debug Tips:** 
*   **Developer Mode:**
    *   "Edit View: Form" to inspect and modify the XML structure directly.
    *   "View Fields" to check field definitions and attributes.
    *   "View Metadata" to see action details and \`external_id\`.
*   **Browser Developer Tools:**
    *   Inspect element to check HTML structure and CSS.
    *   JavaScript console for client-side errors (especially for custom widgets or complex onchange interactions).
    *   Network tab to inspect RPC calls and responses.
*   **Python Debugger:** Use \`pdb\` or your IDE's debugger to step through Python methods (actions, onchanges, computes).
*   **Odoo Server Logs:** Check for tracebacks or error messages, especially for issues related to button actions or data saving.
*   **Logging:** Add \`_logger.info(...)\` statements in Python methods to trace execution flow and variable values.
