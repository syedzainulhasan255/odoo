# Report Generation

## 1. Feature Overview

**Purpose:** 
Odoo's report generation feature allows for the creation of dynamic, data-driven documents, typically in PDF or HTML format. These reports are essential for official business communications and record-keeping, such as generating customer invoices, sales quotations, delivery slips, purchase orders, and custom analytical reports based on data stored within Odoo models.

**Module Location:** 
The core report rendering engine (QWeb) and action definitions are part of the `web` and `base` modules. Specific modules like `account` (for invoices, financial statements), `sale` (for sales orders), `purchase` (for purchase orders), etc., define their own reports and templates.

**Dependencies:** 
*   `ir.actions.report`: The Odoo model that defines a report action.
*   **QWeb Templating Engine:** Odoo's native XML-based templating language used to design the structure and content of reports.
*   **Wkhtmltopdf:** An external command-line utility used by Odoo to convert HTML (generated from QWeb templates) into PDF documents. This must be installed on the Odoo server.
*   Underlying data models that provide the data for the reports.

**User Roles:** 
Report generation is typically used by any user who needs to print, view, or send official business documents or analytical summaries. Access to specific reports can be controlled through user groups and model access rights associated with the `ir.actions.report` record or the underlying data models.

## 2. Technical Details

**Model/View Type:** 
*   **Report Action (`ir.actions.report`):** This model defines the report itself, linking a model to a QWeb template and specifying output format and other settings.
*   **QWeb Template (`ir.ui.view` with `type="qweb"`):** These are XML-based templates that define the structure, layout, and dynamic content of the report.

**XML Structure:** 

1.  **Defining the Report Action (`<report>` tag - shortcut for `ir.actions.report`):**
    This XML record makes the report available in Odoo (e.g., in the "Print" menu of a view).
    ```xml
    <record id="account_invoices" model="ir.actions.report">
        <field name="name">Invoices</field>
        <field name="model">account.move</field> <!-- The model whose records this report is for -->
        <field name="report_type">qweb-pdf</field> <!-- 'qweb-pdf' for PDF, 'qweb-html' for HTML -->
        <field name="report_name">account.report_invoice_with_payments</field> <!-- External ID of the main QWeb template -->
        <field name="report_file">account.report_invoice_with_payments</field> <!-- Usually same as report_name -->
        <field name="print_report_name">(object.state == 'draft') and 'Draft Invoice - %s' % (object.name) or 'Invoice - %s' % (object.name)</field> <!-- Dynamic filename -->
        <field name="paperformat_id" ref="base.paperformat_euro"/> <!-- Optional: specific paper format -->
        <field name="binding_model_id" ref="model_account_move"/> <!-- Makes report available on account.move records -->
        <field name="binding_type">report</field> <!-- Type of binding -->
        <field name="attachment_use" eval="True"/> <!-- If True, generates an attachment on the record -->
        <field name="attachment">(object.state == 'posted') and ('INV-' + object.name + '.pdf')</field> <!-- Dynamic attachment filename -->
    </record>
    ```
    *   `id`: Unique XML ID for the report action.
    *   `string`: User-friendly name of the report displayed in UI.
    *   `model`: The technical name of the Odoo model this report is based on (e.g., `account.move`).
    *   `report_type`: Output format. `qweb-pdf` for PDF, `qweb-html` for HTML.
    *   `name`: The complete external ID (`module.template_id`) of the main QWeb template to render.
    *   `file`: Usually the same as `report_name`.
    *   `print_report_name`: A Python expression to dynamically generate the filename for the downloaded report. `object` refers to the current record.
    *   `paperformat_id`: Reference to an `report.paperformat` record defining page dimensions, orientation, margins, DPI, etc.
    *   `binding_model_id`: Associates the report with a specific model, making it appear in the "Print" menu of that model's views.
    *   `attachment_use` and `attachment`: Control if and how the generated report is saved as an attachment to the record.

2.  **QWeb Template Structure (`<template id="...">`):**
    ```xml
    <template id="report_invoice_with_payments_document"> <!-- Corresponds to report_name -->
        <t t-call="web.html_container"> <!-- Standard wrapper for web pages/reports -->
            <t t-foreach="docs" t-as="o"> <!-- 'docs' is the recordset passed to the report -->
                <t t-call="web.external_layout"> <!-- Standard Odoo header/footer layout -->
                    <div class="page">
                        <h2>Invoice: <span t-field="o.name"/></h2>
                        <p>Partner: <span t-field="o.partner_id.name"/></p>
                        <p>Date: <span t-field="o.invoice_date" t-options='{"widget": "date"}'/> </p>
                        
                        <h3>Invoice Lines</h3>
                        <table class="table table-sm">
                            <thead><tr><th>Product</th><th>Quantity</th><th>Price</th></tr></thead>
                            <tbody>
                                <t t-foreach="o.invoice_line_ids" t-as="line">
                                    <tr>
                                        <td><span t-field="line.product_id.name"/></td>
                                        <td><span t-field="line.quantity"/></td>
                                        <td><span t-field="line.price_unit" t-options='{"widget": "monetary", "display_currency": o.currency_id}'/></td>
                                    </tr>
                                </t>
                            </tbody>
                        </table>
                        
                        <p>Total: <span t-esc="o.amount_total" t-options='{"widget": "monetary", "display_currency": o.currency_id}'/></p>
                        
                        <!-- Example of calling another template -->
                        <!-- <t t-call="account.report_invoice_payment_info"/> -->
                    </div>
                </t>
            </t>
        </t>
    </template>
    ```
    *   `<template id="unique_template_id">`: Defines a QWeb template.
    *   `t-call="web.html_container"`: Essential wrapper for all reports to ensure proper HTML structure and asset loading.
    *   `t-foreach="docs" t-as="doc"` (or `o`): Iterates over the records passed to the report. `docs` is the default variable name for the recordset.
    *   `t-call="web.external_layout"` (or `web.internal_layout`): Applies a standard Odoo document layout (header, footer, company logo). `external_layout` is common for customer-facing documents.
    *   `t-field="object.field_name"`: Renders a field value, applying field-specific formatting and widgets (e.g., for dates, monetary values).
    *   `t-esc="expression"`: Evaluates a Python expression and HTML-escapes the result.
    *   `t-if="condition"`, `t-set="variable" value="expression"`: Conditional rendering and variable setting.

**Python Backend:** 
*   **Data Source:** The records for the report are typically the active records from which the report action was triggered, or records selected by a wizard. These are passed as `docids` to the report processing logic.
*   **`models.AbstractModel`:** To provide custom data or helper functions to your QWeb template, you can create a model inheriting from `models.AbstractModel`. The name of this model must match the `report_name` specified in the `ir.actions.report` record (e.g., `_name = 'report.my_module.my_template_name'`).
*   **`_get_report_values(self, docids, data=None)` method:** This method within your abstract model is responsible for fetching and preparing data for the report.
    *   `docids`: A list of IDs of the main records for which the report is being generated.
    *   `data`: Optional dictionary that can contain additional data passed from a wizard or context.
    *   It must return a dictionary. The keys of this dictionary become available as variables in the QWeb template. A common practice is to include `'docs': self.env['your.model'].browse(docids)` to pass the recordset.
    ```python
    from odoo import models, api

    class MyCustomReport(models.AbstractModel):
        _name = 'report.my_module.report_partner_custom_template' # Must match <report name="...">
        _description = 'My Custom Report Data Provider'

        @api.model
        def _get_report_values(self, docids, data=None):
            report_docs = self.env['res.partner'].browse(docids)
            company_address = self.env.company.partner_id.display_name # Example helper data
            return {
                'doc_ids': docids,
                'doc_model': 'res.partner',
                'docs': report_docs,
                'company_address': company_address,
                'extra_info': 'This report was generated on %s' % fields.Date.today(),
            }
    ```

**Database Impact:** 
Report generation primarily involves **read operations** from the database to fetch the necessary data for display. It generally does not modify data, although certain actions associated with printing (like logging the print date or updating a "printed" status on a record) might be triggered by the same button or workflow that generates the report.

## 3. Visual Documentation

**Mermaid Diagrams:**
```mermaid
graph TD
    A[User Clicks "Print" Button / Triggers Report Action] --> B(ir.actions.report Definition Loaded);
    B --> C[Python: _get_report_values(docids) Called];
    C --> D[Data Fetched from Database (docs, other_data)];
    D --> E[QWeb Template (report_name) Rendered with Data];
    E --> F{Report Type?};
    F -- qweb-pdf --> G[HTML Output Generated by QWeb];
    G --> H[Wkhtmltopdf Converts HTML to PDF];
    H --> I[PDF Document Served to User / Attached];
    F -- qweb-html --> J[HTML Output Served to User];
```

**Screenshots Description & UI Layout:** 
*   **Trigger Points:** Reports are typically accessed via a "Print" dropdown menu on Form or List views of records (e.g., an "Invoice" button on an `account.move` form). They can also be available from specific "Reporting" menus within applications.
*   **Output:** For `qweb-pdf` reports, the output is a PDF document that either downloads directly or opens in a new browser tab/viewer. For `qweb-html`, it's a standard HTML page.
*   **Example PDF:** A typical Odoo PDF report (like an invoice) will have:
    *   A header with the company logo, address, and document title.
    *   Recipient details (e.g., customer address).
    *   Document-specific information (e.g., invoice number, date, due date).
    *   A table of lines (e.g., invoice lines with product, quantity, price).
    *   Totals and other summary information.
    *   A footer with page numbers, company details, or terms and conditions.

## 4. Functionality Description

**Core Features:** 
*   Dynamic generation of PDF and HTML documents from Odoo record data.
*   Flexible and powerful layout design using QWeb templating language, allowing for loops, conditionals, field formatting, and inclusion of other templates.
*   Support for company-specific headers and footers (`web.external_layout` or `web.internal_layout`).
*   Ability to include dynamic data from models and custom Python helper methods.
*   Multilingual support: QWeb templates can be translated like other Odoo views.
*   Configurable paper formats (size, orientation, margins).

**Configuration Options (on `ir.actions.report`):** 
*   `report_type`: `qweb-pdf` (default) or `qweb-html`.
*   `model`: The main model for the report.
*   `name` (`report_name`): The XML ID of the main QWeb template.
*   `paperformat_id`: Reference to a `report.paperformat` record.
*   `attachment_use`: Boolean; if true, the report can be stored as an attachment.
*   `attachment`: Python expression to define the filename of the attachment.
*   `print_report_name`: Python expression for the downloaded filename.
*   `binding_model_id`: Links the report to a model's "Print" menu.
*   `groups_id`: Restricts report visibility to specific user groups.

**Behavior Variations:** 
*   **PDF vs. HTML Output:** Determined by `report_type`.
*   **Paper Format:** Different `report.paperformat` records can be used for different reports or companies.
*   **Custom Filenames:** Dynamically generated filenames for downloads.
*   **Single vs. Batch Reports:** Reports can be designed to process and display a single record or a batch of selected records.
*   **Direct Print / Download / Attachment:** Behavior can vary based on context and configuration.

**Integration Points:** 
*   Most commonly triggered from the "Print" menu on Form and List views.
*   Can be called by server actions or other Python code to generate reports programmatically.
*   Generated reports (especially PDFs) can be automatically attached to records (e.g., an invoice PDF attached to the `account.move` record).
*   Can be used as attachments in emails sent from Odoo.

## 5. Implementation Examples

**XML Configuration:**

*Report Action (Simplified Invoice Report):*
```xml
<record id="action_report_account_invoice_example" model="ir.actions.report">
    <field name="name">Invoice Example</field>
    <field name="model">account.move</field>
    <field name="report_type">qweb-pdf</field>
    <field name="report_name">my_module.report_invoice_document_example</field>
    <field name="report_file">my_module.report_invoice_document_example</field>
    <field name="print_report_name">'INV-' + (object.name or '').replace('/','_')</field>
    <field name="binding_model_id" ref="account.model_account_move"/>
    <field name="binding_type">report</field>
</record>
```

*QWeb Template Snippet (Simplified Invoice):*
```xml
<template id="report_invoice_document_example">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o"> <!-- 'docs' is provided by _get_report_values -->
            <t t-call="web.external_layout"> <!-- Uses company header/footer -->
                <div class="page">
                    <h2>Invoice: <span t-field="o.name"/></h2>
                    <div class="row mt32 mb32">
                        <div class="col-6">
                            <strong>Invoice To:</strong>
                            <address t-field="o.partner_id" t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                        </div>
                        <div class="col-6 text-end">
                            <p><strong>Invoice Date:</strong> <span t-field="o.invoice_date"/></p>
                            <p><strong>Due Date:</strong> <span t-field="o.invoice_date_due"/></p>
                        </div>
                    </div>

                    <table class="table table-sm">
                        <thead>
                            <tr>
                                <th>Description</th>
                                <th class="text-end">Quantity</th>
                                <th class="text-end">Unit Price</th>
                                <th class="text-end">Amount</th>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="o.invoice_line_ids" t-as="line">
                                <tr>
                                    <td><span t-field="line.name"/></td>
                                    <td class="text-end"><span t-field="line.quantity"/></td>
                                    <td class="text-end"><span t-field="line.price_unit"/></td>
                                    <td class="text-end"><span t-field="line.price_subtotal"
                                        t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                                    </td>
                                </tr>
                            </t>
                        </tbody>
                    </table>

                    <div class="row justify-content-end">
                        <div class="col-4">
                            <table class="table table-sm">
                                <tr class="border-black">
                                    <td><strong>Total</strong></td>
                                    <td class="text-end">
                                        <span t-field="o.amount_total"
                                            t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                                    </td>
                                </tr>
                            </table>
                        </div>
                    </div>
                </div>
            </t>
        </t>
    </t>
</template>
```

**Python Code (AbstractModel for Report Data):**
```python
from odoo import models, api, fields

class AccountInvoiceReportExample(models.AbstractModel):
    _name = 'report.my_module.report_invoice_document_example' # Corresponds to template ID
    _description = 'Example Invoice Report Data Provider'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['account.move'].browse(docids)
        
        company = self.env.company
        
        return {
            'doc_ids': docids,
            'doc_model': 'account.move',
            'docs': docs,
            'company': company,
            'current_date_formatted': fields.Date.today().strftime('%Y-%m-%d'),
            # Add any other helper functions or data needed in the QWeb template
            # e.g., a method to format addresses, calculate specific values, etc.
        }
```

**JavaScript (if applicable):**
[Placeholder: JavaScript is generally not directly involved in the server-side generation of QWeb PDF reports. If `report_type="qweb-html"`, JavaScript can be used within the QWeb template just like in any other webpage for client-side interactivity. For PDF reports, any JS in the template is typically executed by Wkhtmltopdf during rendering, but complex JS manipulations are unreliable.]

## 6. Customization Guide

**Common Modifications:** 
*   **Layout Changes:** Modifying the QWeb template (`<template>`) HTML structure, adding/removing/rearranging elements, changing CSS classes.
*   **Adding/Removing Data:**
    *   Displaying new fields from the main model (`docs`) using `t-field` or `t-esc`.
    *   Adding new data by extending the dictionary returned by the `_get_report_values` Python method.
    *   Adding related model data (e.g., `o.partner_id.street`).
*   **Paper Format:** Creating or modifying `report.paperformat` records (accessible via Settings -> Technical -> Reporting -> Paper Format) to change page size, orientation, margins, header/footer spacing, and DPI.
*   **Translations:** Translating static text within QWeb templates using standard Odoo translation mechanisms (`.po` files).
*   **Report Filename:** Customizing the downloaded filename using the `print_report_name` field on `ir.actions.report`.

**Extension Points:** 
*   **QWeb Template Inheritance:** Use `<xpath expr="..." position="...">` to inherit and modify existing QWeb report templates. This is the preferred method for customizing standard reports.
*   **Overriding `_get_report_values`:** Inherit the `models.AbstractModel` associated with a report to modify its `_get_report_values` method, allowing you to add new data to `docargs` or change existing data passed to the QWeb template.
*   **Creating New Reports:** Define a new `ir.actions.report` record and its corresponding QWeb templates and (optionally) an `models.AbstractModel` for data preparation.

**Best Practices:** 
*   Utilize Odoo's standard layouts like `web.external_layout` or `web.internal_layout` as a base for consistency (headers, footers, company logo).
*   Keep QWeb templates readable and well-structured. Use `t-call` to include sub-templates for reusable components.
*   Move complex data processing or formatting logic into Python helper methods within your `_get_report_values` or on the related models, rather than embedding it directly and complexly in QWeb.
*   Optimize data fetching in `_get_report_values`. Avoid unnecessary browsing or searching within loops.
*   Use CSS for styling within QWeb templates, either inline or by referencing external CSS files included in the report assets.
*   Ensure all user-facing text in templates is translatable (e.g., use `t-esc` for dynamic data, wrap static text in `_()` in Python or ensure it's part of translatable QWeb).

**Pitfalls to Avoid:** 
*   **Complex Logic in QWeb:** Overly complex Python-like expressions within `t-if`, `t-set`, or `t-esc` can make templates hard to read, debug, and maintain.
*   **Hardcoding Text:** Avoid hardcoding translatable text directly in templates; use field labels or `t-esc` with translatable Python strings.
*   **Inefficient Data Queries:** Fetching too much data or performing inefficient loops within `_get_report_values` can lead to very slow report generation.
*   **Wkhtmltopdf Issues:** Ensure Wkhtmltopdf is correctly installed on the server and is a compatible version. Some complex CSS or JavaScript might not render perfectly in PDF.
*   **Direct SQL Queries:** Avoid raw SQL queries in report methods if the ORM can achieve the same, for security and maintainability.

## 7. Testing & Troubleshooting

**Test Scenarios:** 
*   Generate the report for a single record and for multiple records (if applicable).
*   Verify all data fields appear correctly and are formatted as expected (dates, numbers, currencies).
*   Check headers, footers, and page numbering for accuracy and consistency.
*   Test the PDF output in different PDF viewers (e.g., Adobe Reader, browser built-in viewers) to ensure consistent rendering.
*   If the report is multilingual, test it in different languages.
*   Verify the chosen paper format and margins are correctly applied.
*   Test with various data scenarios (e.g., records with missing optional fields, records with long text in fields, zero-value amounts).

**Common Issues:** 
*   **Report Not Found/Action Error:** Check that the `report_name` in `ir.actions.report` correctly matches the XML ID of the main QWeb template.
*   **QWeb Template Errors:** Syntax errors in QWeb (e.g., unclosed tags, incorrect `t-` directive usage). Variable names in the template not matching those passed from `_get_report_values`.
*   **Wkhtmltopdf Errors:** Ensure Wkhtmltopdf is installed, in the system PATH, and is a version compatible with your Odoo version. Check server logs for specific Wkhtmltopdf error messages.
*   **Data Not Appearing:** Verify the data is being fetched correctly in `_get_report_values` and passed to the `docargs` dictionary (usually under the `docs` key). Check field names in the QWeb template match the model's field names.
*   **Styling/Layout Issues:** CSS conflicts, incorrect HTML structure, or Wkhtmltopdf limitations in rendering complex CSS.

**Debug Tips:** 
*   **Odoo Server Logs:** The primary source for backend errors, including Python errors in `_get_report_values` and Wkhtmltopdf errors. Increase log verbosity if needed.
*   **`--dev=xml` Server Option:** When running Odoo with this option, QWeb rendering errors can sometimes provide more detailed tracebacks directly in the browser if the report fails to generate.
*   **Switch to HTML Output:** Temporarily change the `report_type` in `ir.actions.report` from `qweb-pdf` to `qweb-html`. This will render the report as a plain HTML page in the browser, allowing you to use browser developer tools to inspect the generated HTML structure, debug CSS, and see JavaScript errors (if any). The `?debug=assets` or `?debug=qweb` URL parameters can also be helpful here.
*   **Python Debugger:** Use `pdb.set_trace()` or an IDE debugger within the `_get_report_values` method to inspect the `docids`, `data`, and the content of the dictionary being returned.
*   **Minimal Template:** If facing complex QWeb issues, try simplifying the template to its bare minimum and gradually add back sections to isolate the problematic part.
*   **Check Paper Format:** Incorrect paper format settings can lead to content being cut off or poorly scaled.
