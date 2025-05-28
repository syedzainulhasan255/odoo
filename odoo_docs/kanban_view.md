# Kanban View

## 1. Feature Overview

**Purpose:** 
[Placeholder: Display records as cards in a board-like interface, often grouped by stages or categories. Provides a visual way to manage workflows (e.g., sales pipeline, project tasks) and allows for drag-and-drop interaction.]

**Module Location:** 
[Placeholder: \`web\` (core Kanban view functionality), and various modules that use Kanban views (e.g., \`crm\`, \`project\`, \`mrp`).]

**Dependencies:** 
[Placeholder: View architecture, ORM. Often used with fields that represent stages or groups.]

**User Roles:** 
[Placeholder: Users who manage records in a workflow or prefer a visual card-based overview.]

## 2. Technical Details

**Model/View Type:** 
[Placeholder: \`ir.ui.view\`, type \`kanban\`.]

**XML Structure:** 
[Placeholder: \`<kanban default_group_by="stage_id"> <field name="field1"/> <field name="field2"/> <templates><t t-name="kanban-box"> <!-- QWeb template for card layout --> <div class="oe_kanban_card"> ... </div> </t></templates> </kanban>\`. Key elements include \`<field>\` (fields available for grouping or display), \`<templates>\` (containing QWeb for card rendering), and attributes like \`default_group_by\`, \`quick_create\`.]

**Python Backend:** 
[Placeholder: Model's \`search_read\` is often used to fetch data. \`fields_view_get\` for view definition. Methods on the model might be called by buttons or actions within Kanban cards. Grouping is handled by the ORM.]

**Database Impact:** 
[Placeholder: Reads data for display. Drag-and-drop actions typically update the grouping field (e.g., \`stage_id\`) on the record.]

## 3. Visual Documentation

**Mermaid Diagrams:**
\`\`\`mermaid
graph TD
    A[User Opens Kanban View] --> B(Kanban Renders with Cards in Columns);
    B --> C{User Interacts};
    C -- Drags Card --> D[Record's Grouping Field Updated];
    D --> B;
    C -- Clicks Card/Button --> E[Action Triggered / Form View Opens];
\`\`\`
[Placeholder: Illustrate the structure of a Kanban board and common user interactions like drag-and-drop.]

**Screenshots Description:** 
[Placeholder: Show a typical Odoo Kanban view: columns representing stages (e.g., "New", "In Progress", "Done"), cards within columns representing records. Highlight features like progress bars, avatars, and quick create options within columns.]

**UI Layout:** 
[Placeholder: Columns represent groups (often stages). Cards within columns display summarized record information. May include a "Quick Create" option at the top of columns. Control panel often includes search filters and group by options.]

## 4. Functionality Description

**Core Features:** 
[Placeholder: Visualize records as cards. Group cards into columns based on a field (e.g., stage, priority). Drag-and-drop cards between columns to update their group. Quick create records directly within columns. Display key information and progress on cards.]

**Configuration Options:** 
[Placeholder: \`default_group_by\` (field to group by default). \`quick_create="true/false"\` (enable/disable quick create). \`archivable="true/false"\`. \`group_create\` / \`group_edit\` / \`group_delete\` (allow column management). QWeb templates define card layout.]

**Behavior Variations:** 
[Placeholder: Appearance of cards can vary greatly based on QWeb template. Different actions available on cards. Behavior of drag-and-drop (e.g., restrictions, automatic updates).]

**Integration Points:** 
[Placeholder: Tightly integrated with the underlying model and its fields. Clicking a card often opens the Form view for that record. Can trigger server actions or Python methods from within cards.]

## 5. Implementation Examples

**XML Configuration:**
\`\`\`xml
<!-- Kanban View Definition -->
<record id="view_project_task_kanban_example" model="ir.ui.view">
    <field name="name">project.task.kanban.example</field>
    <field name="model">project.task</field> <!-- Assuming project.task model -->
    <field name="arch" type="xml">
        <kanban default_group_by="stage_id" class="o_kanban_small_column" quick_create="true">
            <field name="stage_id"/>
            <field name="name"/>
            <field name="user_ids"/> <!-- Changed from user_id to user_ids for Many2many -->
            <field name="priority"/>
            <field name="color"/> <!-- For color tags on kanban cards -->
            <templates>
                <t t-name="kanban-box">
                    <div t-attf-class="oe_kanban_color_#{kanban_getcolor(record.color.raw_value)} oe_kanban_card oe_kanban_global_click">
                        <div class="o_kanban_record_top">
                            <div class="o_kanban_record_headings">
                                <strong class="o_kanban_record_title"><field name="name"/></strong>
                            </div>
                            <field name="priority" widget="priority"/>
                        </div>
                        <div class="o_kanban_record_body">
                            <!-- Display more fields here -->
                            <field name="project_id"/>
                        </div>
                        <div class="o_kanban_record_bottom">
                            <div class="oe_kanban_bottom_left">
                                <field name="user_ids" widget="many2many_avatar_user"/>
                            </div>
                            <div class="oe_kanban_bottom_right">
                                <!-- Progress bar or other indicators -->
                            </div>
                        </div>
                    </div>
                </t>
            </templates>
        </kanban>
    </field>
</record>
\`\`\`

**Python Code:**
\`\`\`python
# Show related Python model/method implementations
from odoo import models, fields, api

class ExampleProjectTaskKanban(models.Model):
    _inherit = 'project.task' # Assuming project.task is the model

    # stage_id = fields.Many2one('project.task.type', string='Stage')
    # name = fields.Char(string="Task Name")
    # user_ids = fields.Many2many('res.users', string='Assignees') # Corrected to user_ids
    # priority = fields.Selection([('0','Low'), ('1','Normal'), ('2','High')], string='Priority')
    # color = fields.Integer('Color Index')
    # project_id = fields.Many2one('project.project', string='Project')

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        # Custom method to define order or content of stages/columns in Kanban
        # This is an example, often stage model's _read_group is used by default
        return stages.search([], order=order)

    # Methods called by buttons inside Kanban cards would be defined here
    def action_view_details(self):
        self.ensure_one()
        # Return action to open form view or other relevant action
        return {
            'type': 'ir.actions.act_window',
            'res_model': self._name,
            'view_mode': 'form',
            'res_id': self.id,
            'target': 'current', # or 'new' for a dialog
        }

# Note: Drag-and-drop updates the 'stage_id' (or the default_group_by field) directly via ORM write calls.
# The \`quick_create="true"\` attribute uses the model's \`create()\` method.
\`\`\`

**JavaScript (if applicable):**
\`\`\`javascript
// Example of JS for custom Kanban interactions (e.g., custom quick create)
// odoo.define('your_module.kanban_view_customization', function (require) {
// "use strict";
//
// var KanbanController = require('web.KanbanController');
// var KanbanRenderer = require('web.KanbanRenderer');
//
// KanbanController.include({
//     _onAddRecordToColumn: function (ev) {
//         // Custom logic for quick create
//         this._super.apply(this, arguments);
//     },
// });
//
// KanbanRenderer.include({
//     _renderCard: function (record) {
//         var $card = this._super.apply(this, arguments);
//         // Custom modifications to the card after rendering
//         return $card;
//     }
// })
// });
\`\`\`

## 6. Customization Guide

**Common Modifications:** 
[Placeholder: Changing the layout of cards using QWeb. Adding new fields to cards. Customizing the behavior of drag-and-drop (e.g., validation). Adding buttons or links to cards. Modifying how columns are defined or ordered.]

**Extension Points:** 
[Placeholder: Inheriting the Kanban view XML to modify \`<field>\` elements or the QWeb template within \`<templates>\`. Overriding model methods related to Kanban actions (e.g., \`write\` for drag-and-drop if custom logic is needed, or methods called by buttons). Creating custom JavaScript (Owl components) for highly interactive Kanban features.]

**Best Practices:** 
[Placeholder: Keep card information concise and relevant. Optimize QWeb templates for performance. Ensure drag-and-drop interactions are smooth and provide clear feedback. Use existing Odoo CSS classes for consistent styling.]

**Pitfalls to Avoid:** 
[Placeholder: Overloading cards with too much information. Complex QWeb logic slowing down rendering. JavaScript customizations that break core Kanban behavior. Not testing responsiveness on different screen sizes.]

## 7. Testing & Troubleshooting

**Test Scenarios:** 
[Placeholder: 
1. Verify cards render correctly in each column.
2. Test drag-and-drop of cards between columns and verify data updates.
3. Test quick create functionality in columns (if enabled).
4. Test any buttons or actions within cards.
5. Verify grouping and sorting options.
6. Check responsiveness of the Kanban view.]

**Common Issues:** 
[Placeholder: Cards not displaying correctly (check QWeb template syntax, field names). Drag-and-drop not working (check \`default_group_by\` field, model write permissions). Performance issues with many cards or complex QWeb. JavaScript errors in the browser console.]

**Debug Tips:** 
[Placeholder: Activate developer mode to inspect Kanban view XML and QWeb templates. Use browser developer tools to inspect card HTML/CSS and debug JavaScript. Check server logs for errors during data fetching or updates. For Owl-based Kanban elements, use the Owl DevTools extension.]
