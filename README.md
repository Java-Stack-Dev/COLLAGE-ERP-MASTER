# COLLAGE-ERP-MASTER
Attendance Management Enhancements:

Implement SMS notifications to parents for student absences, including storing parent phone numbers and integrating with an SMS API.
Display the message sent status on the user dashboard.
Create a filterable student attendance list with options to filter by date, course, semester, and status.
Allow faculty to view and edit attendance, while restricting modifications for student, course, and date.
Enable administrators to monitor attendance.
Add functionality to export the attendance list as PDF and Excel files.
Admin Panel Customization:

Retain and enhance the Django admin panel (/admin) to manage users (students, faculty, staff), departments, courses, classes, and timetables.
Implement access control mechanisms to grant or revoke user permissions.
Provide options to view system logs and generate various reports (attendance, marks, fees).
Library Management Module:

Add a new library module to manage books, including features for:
Library staff to add, edit, remove, and categorize books by subject/author.
Tracking book issuance and returns, and managing fines.
Students and faculty to search the book catalog, view their borrowing history, and request reservations.
Integrate borrowed book records with the student details view in the existing system.

##################################################################################################################################


1. Attendance Management Enhancements

SMS Notifications for Absences:
Model Modification: Add a parent_phone field (CharField, max_length=15, blank=True, null=True) to the Student model in info/models.py to store the parent's contact number.
View Modification (confirm): In the confirm view function in info/views.py (where attendance is recorded), add logic to:
Identify students marked as absent.
Retrieve the parent_phone number for each absent student.
Call a function send_absence_sms(phone_number, student_name, course_name, date) to send an SMS notification to the parent.
Update the corresponding Attendance record's message_sent field to True after successfully sending the SMS.
SMS Function (send_absence_sms): Create this function in info/views.py to:
Accept the parent's phone_number, student's name, course_name, and date as arguments.
Integrate with an SMS API (e.g., Twilio, Plivo) using the appropriate library. This involves installing the library (e.g., pip install twilio) and using its functions to send the SMS. You'll need to obtain API credentials from the chosen provider.
Include error handling to manage potential issues during SMS sending.
Dashboard Display:
Add a message_sent field (BooleanField, default=False) to the Attendance model in info/models.py to track whether a notification has been sent for an absence.
Modify the index view (student dashboard) in info/views.py to retrieve attendance records for the logged-in student, including the message_sent status.
Update the student dashboard template (info/homepage.html) to display the message_sent status (e.g., "Message Sent" or "No Message Sent") for each absence record. Adapt the HTML structure to fit your existing template.
Filterable Attendance List:
Admin Customization: In info/admin.py, create a custom ModelAdmin class for the Attendance model (e.g., AttendanceAdmin).
Use list_display to specify the fields to display in the list view (e.g., student, course, date, status).
Use list_filter to enable filtering by date, course, semester (if accessible through a relationship like student__class_id__sem), and status. Adjust relationship traversal as needed.
Use search_fields to allow searching by student name and course name (e.g., student__name, course__name).
Use readonly_fields to make student, course, and date read-only in the edit view, preventing accidental modifications by faculty.
Permissions: In the AttendanceAdmin class, implement the has_change_permission method to control editing access:
Allow superusers (admins) to always edit.
Grant editing permission to faculty members who have a specific permission (e.g., info.change_attendance). You might need to create this custom permission if it doesn't exist using a data migration.
Export Functionality: In the AttendanceAdmin class, add admin actions to export the filtered attendance list as PDF and Excel files:
Create functions export_as_pdf and export_as_excel. These functions should:
Accept the request and queryset (selected records) as arguments.
Use a library like reportlab or xhtml2pdf (for PDF) and openpyxl or xlsxwriter (for Excel) to generate the file. Remember to install these libraries (pip install ...).
Format the data from the queryset appropriately for the chosen file format.
Return an HttpResponse with the file content and appropriate content type and disposition headers.
Add both functions to the actions list in the AttendanceAdmin class to make them available in the admin interface.
2. Admin Panel Customization

Model Registration: In the admin.py file of each app (e.g., info, library), register all relevant models using admin.site.register(). For example, in info/admin.py, register Student, Teacher, Dept, Course, Class, etc. You can use natural language for this: "Register the following models in the Django admin panel: [list of models]. Import the models from .models."
Admin Model Customization (Optional but Recommended): For each model, create a custom ModelAdmin class to tailor the admin interface. This includes:
list_display: Control which fields are displayed in the list view.
search_fields: Enable searching by specific fields.
list_filter: Add filters to the list view.
ordering: Specify default ordering of records.
inlines: Manage related objects within the same admin page (e.g., display a student's attendance records as an inline on the student edit page).
Custom forms: Fine-tune the fields shown in add/edit views and add validation.
User Management: Django's built-in authentication system (accessible under "Users" in the admin panel) handles adding, editing, and deleting users.
Permissions: Utilize Django's groups and permissions to control access levels. Assign users to groups (e.g., student, faculty, staff, admin) and grant specific permissions to each group. Manage permissions through the admin panel (Users > [select user] > Permissions).
Reporting and Logs (Advanced):
Custom Admin Actions: For reports beyond the basic filtering provided, create custom admin actions. These are functions added to a ModelAdmin class that can be triggered from the list view. The actions should:
Retrieve the relevant data (using the selected queryset or querying the database).
Format the data.
Generate the report (e.g., using a library like reportlab for PDF).
Return an HttpResponse with the report file.
System Logs: Django doesn't have built-in system logging within the admin panel. Consider integrating a separate logging solution (or writing custom views) to display log data. Use Django's logging framework to capture events and present them in a user-friendly way.
3. Library Management Module

App Creation: Create a new Django app named library using the command python manage.py startapp library. You may need to activate your virtual environment first.
Models (library/models.py): Define the core data models for the library:
Book: Fields like title (CharField), author (CharField), subject (CharField), isbn (CharField, unique=True), total_copies (IntegerField), available_copies (IntegerField).
BorrowRecord: Fields to track borrowings, including:
book (ForeignKey to Book).
borrower (ForeignKey to Student, null=True, blank=True) and/or borrower_teacher (ForeignKey to Teacher, null=True, blank=True) to handle borrowing by either students or teachers.
borrow_date (DateField, auto_now_add=True).
return_date (DateField, null=True, blank=True).
due_date (DateField).
fine_amount (DecimalField).
Admin Interface (library/admin.py): Create ModelAdmin classes for Book and BorrowRecord to manage them through the admin panel:
Customize list_display, search_fields, and list_filter to make the interface user-friendly.
Consider using inlines to manage related records (e.g., display borrow records for a book on the book's edit page).
Student/Faculty Interface (Views and URLs): Create views in library/views.py and define URL patterns in library/urls.py for students and faculty to interact with the library:
book_list: Display the book catalog with search and filtering.
borrowing_history: Show a user's borrowing history. Distinguish between students and teachers to query the correct BorrowRecord entries.
request_reservation (Optional): Implement book reservations if desired.
Create corresponding HTML templates in library/templates/library/ to render these views.
Include the library app's URLs in the project's main urls.py file.
Integration with Student Details: In the info app (where student details are assumed to be), modify the view that displays student information to also retrieve and display the student's borrowing history from the library app. Update the corresponding template to show this information.
Database Migrations: After defining the models, run python manage.py makemigrations library and python manage.py migrate to create the necessary tables in the database.
