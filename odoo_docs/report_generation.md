# Report Generation

## 1. Feature Overview

**Purpose:** 
[Placeholder: Generate printable documents (typically PDFs) like invoices, sales orders, delivery slips, or custom reports based on Odoo data.]

**Module Location:** 
[Placeholder: `web`, `base`, and specific modules that define reports (e.g., `account` for invoices, `sale` for sales orders).]

**Dependencies:** 
[Placeholder: `ir.actions.report`, QWeb templating engine, Wkhtmltopdf utility (for PDF generation).]

**User Roles:** 
[Placeholder: Users who need to print or view official documents or analytical reports. Access may be controlled per report or model.]

## 2. Technical Details

**Model/View Type:** 
[Placeholder: `ir.actions.report` defines the report action. QWeb templates (`ir.ui.view` with type `qweb`) define the report's structure and content.]

**XML Structure:** 
[Placeholder: `<report id="action_report_myspecificmodel" string="My Report" model="my.specific.model" report_type="qweb-pdf" name="my_module.report_myspecificmodel_template" file="my_module.report_myspecificmodel_template"/>`. QWeb templates use standard XML/HTML syntax with QWeb directives (`t-foreach`, `t-if`, `t-esc`, `t-field`).]

**Python Backend:** 
[Placeholder: Models provide data to reports. Custom Python methods can be called from QWeb templates (e.g., for complex data processing or formatting) by adding them to `docargs` in the `_get_report_values` method of the report's associated model or a custom `report.abstract_model`.]

**Database Impact:** 
[Placeholder: Reads data from various tables to populate the report. Does not typically write data, but the action of printing might trigger status changes on some models (e.g., marking an invoice as printed).]

## 3. Visual Documentation

**Mermaid Diagrams:**
```mermaid
graph TD
    A[User Clicks Print/Report Button] --> B(ir.actions.report Triggered);
    B --> C{Report Type? (PDF/HTML)};
    C -- PDF --> D[QWeb Template Rendered to HTML];
    D --> E[HTML Converted to PDF via Wkhtmltopdf];
    E --> F[PDF Downloaded/Displayed];
    C -- HTML --> G[QWeb Template Rendered to HTML];
    G --> H[HTML Displayed];
```
[Placeholder: Show the general flow of report generation from user action to document output.]

**Screenshots Description:** 
[Placeholder: Show an example of a "Print" menu in Odoo (e.g., on an invoice or sales order) and an example of a generated PDF report, highlighting key sections like header, footer, data table, and totals.]

**UI Layout:** 
[Placeholder: Reports are typically accessed via a "Print" button or menu item on relevant views or from a dedicated "Reporting" menu. The output is a PDF or HTML document.]

## 4. Functionality Description

**Core Features:** 
[Placeholder: Generate PDF and HTML documents from Odoo data. Use QWeb templates for flexible layout and data presentation. Support for headers, footers, and dynamic content. Can be multilingual.]

**Configuration Options:** 
[Placeholder: `report_type` (`qweb-pdf`, `qweb-html`). `model` (the primary model for the report). `name` (unique template name). `paperformat_id` (specifies paper size, orientation, margins). `attachment_use` (to save as attachment). `print_report_name` (filename expression).]

**Behavior Variations:** 
[Placeholder: PDF vs. HTML output. Different paper formats. Custom filenames for downloaded reports. Reports can be designed for single records or multiple records.]

**Integration Points:** 
[Placeholder: Called from "Print" buttons in views. Can be triggered by server actions. Output can be attached to records or sent by email.]

## 5. Implementation Examples

**XML Configuration:**
```xml
<!-- Report Action -->
<record id="action_report_partner_custom" model="ir.actions.report">
    <field name="name">Custom Partner Report</field>
    <field name="model">res.partner</field>
    <field name="report_type">qweb-pdf</field>
    <field name="report_name">my_module.report_partner_custom_template</field>
    <field name="report_file">my_module.report_partner_custom_template</field>
    <field name="binding_model_id" ref="model_res_partner"/>
    <field name="binding_type">report</field>
</record>

<!-- QWeb Template -->
<template id="report_partner_custom_template">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="doc"> <!-- docs is a list of records -->
            <t t-call="web.external_layout"> <!-- Standard Odoo layout -->
                <div class="page">
                    <h2>Report for <span t-field="doc.name"/></h2>
                    <p>Email: <span t-field="doc.email"/></p>
                    <p>Phone: <span t-field="doc.phone"/></p>
                    <!-- Add more fields and custom layout -->
                </div>
            </t>
        </t>
    </t>
</template>
```

**Python Code:**
```python
# Show related Python model/method implementations
from odoo import models, api

class CustomPartnerReport(models.AbstractModel):
    _name = 'report.my_module.report_partner_custom_template' # Matches template name
    _description = 'Custom Partner Report Handler'

    @api.model
    def _get_report_values(self, docids, data=None):
        # docids: list of IDs of records for which the report is printed
        # data: additional data passed from wizards or context
        
        partners = self.env['res.partner'].browse(docids)
        
        return {
            'doc_ids': docids,
            'doc_model': 'res.partner',
            'docs': partners, # 'docs' is the variable name QWeb expects for the records
            'custom_data': 'This is some extra data for the report',
            # Add any other helper functions or data needed in the QWeb template
        }

# No specific code needed in res.partner unless it's providing specific helper methods
# for this report.
```

**JavaScript (if applicable):**
[Placeholder: Generally not applicable for server-side QWeb report generation. JS might be used if the report is `qweb-html` and requires client-side interactions, but this is less common for printable documents.]

## 6. Customization Guide

**Common Modifications:** 
[Placeholder: Changing the layout of QWeb templates. Adding or removing fields. Adding custom data or calculations via `_get_report_values`. Modifying paper format. Translating reports.]

**Extension Points:** 
[Placeholder: Inheriting QWeb templates to modify specific parts. Overriding `_get_report_values` to add or change data. Creating new `ir.actions.report` records for new reports.]

**Best Practices:** 
[Placeholder: Use standard Odoo layouts (`web.html_container`, `web.external_layout`) for consistency. Keep QWeb templates readable and well-structured. Optimize data fetching in `_get_report_values` for performance. Use CSS for styling within QWeb.]

**Pitfalls to Avoid:** 
[Placeholder: Complex logic directly in QWeb templates (move to Python helpers). Hardcoding text (use `t-esc` or `t-field` for translatable content). Not testing with different data scenarios (e.g., missing fields, long text). Inefficient queries in `_get_report_values` causing slow report generation.]

## 7. Testing & Troubleshooting

**Test Scenarios:** 
[Placeholder: 
1. Generate the report for a single record and multiple records.
2. Verify all data appears correctly and is formatted as expected.
3. Check headers, footers, and page numbering.
4. Test PDF output in different PDF viewers.
5. Test with different languages if applicable.
6. Verify paper format and margins.]

**Common Issues:** 
[Placeholder: Report not found (check action and template names). QWeb template errors (check syntax, variable names). Wkhtmltopdf errors (ensure it's installed and configured correctly). Data not appearing (check `_get_report_values` and QWeb field access). Styling issues (debug CSS within QWeb or linked stylesheets).]

**Debug Tips:** 
[Placeholder: Run Odoo with `--dev=xml` to see QWeb rendering errors more clearly. Temporarily change `report_type` to `qweb-html` to view the raw HTML output in the browser and debug layout/styling. Add print statements or use a debugger in `_get_report_values`. Check server logs for Wkhtmltopdf errors.]
