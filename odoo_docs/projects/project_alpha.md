# Project Alpha Documentation

## 1. Project Overview

**Purpose:** 
[Placeholder: Describe the main goal and functionality of Project Alpha. e.g., "To create a custom module for managing internal training programs, including course scheduling, trainee enrollment, and progress tracking."]

**Key Features:**
*   [Placeholder: e.g., Course Management]
*   [Placeholder: e.g., Trainee Enrollment]
*   [Placeholder: e.g., Schedule Display]
*   [Placeholder: e.g., Progress Reporting]

**Target Users:**
[Placeholder: e.g., HR department, Employees, Department Managers]

## 2. Odoo Features Used

This project leverages several core Odoo features. For detailed information on each feature, please refer to their respective documentation files:

*   **Form Views:** Used for detailed views of Courses, Trainees, Enrollments. (See `../form_view.md`)
*   **Tree Views:** Used for listing Courses, Trainees, and Enrollments. (See `../tree_view.md`)
*   **Search Filters:** Implemented for finding specific Courses, Trainees, or filtering Enrollments by status. (See `../search_filters.md`)
*   **Action Buttons:** Used on forms for actions like "Enroll in Course", "Mark as Completed", "Generate Certificate". (See `../action_buttons.md`)
*   **Kanban Views:** Used to display available courses and trainee progress through different stages. (See `../kanban_view.md`)
*   **Wizards:** Used for batch enrollment or for a multi-step course creation process. (See `../wizard_forms.md`)
*   **Report Generation:** Used for generating training certificates and attendance reports. (See `../report_generation.md`)
*   **Field Widgets:** Custom widgets might be used for displaying schedules or specialized progress indicators. (See `../field_widgets.md`)

## 3. Data Models Involved

[Placeholder: List the main Odoo models created or extended for this project.]
*   `training.course`: Stores information about available courses.
    *   `name` (Char)
    *   `description` (Html)
    *   `duration_hours` (Float)
    *   `instructor_id` (Many2one to `res.partner`)
*   `training.trainee`: Stores information about individuals undergoing training.
    *   `partner_id` (Many2one to `res.partner`)
    *   `enrollment_ids` (One2many to `training.enrollment`)
*   `training.enrollment`: Links trainees to courses.
    *   `course_id` (Many2one to `training.course`)
    *   `trainee_id` (Many2one to `training.trainee`)
    *   `enrollment_date` (Date)
    *   `status` (Selection: e.g., 'enrolled', 'in_progress', 'completed', 'cancelled')
    *   `completion_date` (Date)

## 4. API Usage

**Data Exposure:**
[Placeholder: This project will expose training course catalog and trainee progress via a custom REST API endpoint.]
*   Endpoint: `/api/training/courses` (GET - List available courses)
*   Endpoint: `/api/training/trainee/{trainee_id}/progress` (GET - Get progress for a specific trainee)

**Data Consumption/Updates (via standard Odoo RPC):**
[Placeholder: Details on how external systems might interact with this project's data via Odoo APIs. e.g., An external HR system might update trainee information or enroll users in courses via XML-RPC or JSON-RPC if Project Alpha's models are exposed.]

Refer to the main `../api_documentation.md` for general Odoo API practices.

## 5. Code Examples

[Placeholder: Provide snippets of XML or Python code that are specific to Project Alpha, demonstrating how the Odoo features are configured or extended for this project's requirements.]

**Example: Custom Action Button on `training.course` form view:**
\`\`\`xml
<!-- In training_course_views.xml -->
<record id="view_training_course_form_extended" model="ir.ui.view">
    <field name="name">training.course.form.extended</field>
    <field name="model">training.course</field>
    <field name="inherit_id" ref="original_module.view_training_course_form"/> <!-- If inheriting -->
    <field name="arch" type="xml">
        <xpath expr="//header" position="inside">
            <button name="action_schedule_course" string="Schedule Course" type="object" class="oe_highlight"/>
        </xpath>
    </field>
</record>
\`\`\`

**Example: Python method for the action button:**
\`\`\`python
# In training_course.py
from odoo import models, fields

class TrainingCourse(models.Model):
    _name = 'training.course'
    # ... other fields ...

    def action_schedule_course(self):
        self.ensure_one()
        # Logic to open a scheduling wizard or perform scheduling actions
        # For example, open a wizard:
        # return {
        #     'type': 'ir.actions.act_window',
        #     'name': 'Schedule Course Wizard',
        #     'res_model': 'training.schedule.wizard',
        #     'view_mode': 'form',
        #     'target': 'new',
        #     'context': {'default_course_id': self.id}
        # }
        pass
\`\`\`
