# Contract Monthly Claim System (CMCS) - PROG6212 Final POE

## Project Overview
A comprehensive web-based system for managing monthly contract claims for lecturers at an educational institution. The system implements a multi-role workflow with HR management, lecturer claim submissions, and a two-tier approval process.

## 🆕 Updates from Part 2 to Part 3

### Based on Lecturer Feedback:

1. **HR Super User Role Implemented**
   - HR can now create, edit, and manage all user accounts
   - HR sets hourly rates for all lecturers
   - Public registration completely disabled
   - All users must be created by HR department

2. **Lecturer View Enhancements**
   - Hourly rate automatically pulled from HR-set values
   - Auto-calculation of claim amounts (Hours × Rate)
   - Cannot manually edit hourly rate
   - Enhanced validation for 180-hour monthly maximum
   - Real-time calculation display

3. **Session Management (PROG6212 Requirement)**
   - Comprehensive session implementation throughout application
   - User data stored in sessions during login
   - Session validation on all protected pages
   - Role-specific session data

4. **Authorization & Security**
   - `[Authorize]` attributes on all controllers
   - Role-based access control implemented
   - Users cannot access unauthorized pages
   - Automatic redirection to login for unauthorized access

5. **Report Generation with LINQ**
   - Monthly claims reports using LINQ queries
   - Lecturer performance reports with statistics
   - HTML reports downloadable
   - Department-wise breakdowns using LINQ GroupBy

6. **Database & Entity Framework**
   - EF Core with SQL Server
   - Code-First approach with automatic migrations
   - Database seeding with default users

## 🎯 Key Features (Part 3)

### HR Dashboard (Super User)
- ✅ Create new users with all details (name, email, hourly rate, etc.)
- ✅ Edit existing user information
- ✅ Activate/deactivate user accounts
- ✅ Reset user passwords
- ✅ Generate comprehensive reports using LINQ
- ✅ Download reports (HTML format)
- ✅ View system-wide statistics

### Lecturer Features
- ✅ Auto-populated hourly rate from HR settings
- ✅ Auto-calculation: Total = Hours × HR-set Rate
- ✅ Maximum 180 hours per month validation
- ✅ Track claims through approval workflow
- ✅ View claim history and status
- ✅ Upload supporting documents

### Programme Coordinator Features
- ✅ Review submitted claims
- ✅ Approve/reject claims (first level)
- ✅ Forward approved claims to Academic Manager
- ✅ Session-based authentication

### Academic Manager Features
- ✅ Final approval of claims
- ✅ View all approved claims
- ✅ Generate reports
- ✅ Session-based authentication

## 🔐 Authentication & Authorization

### Session Implementation
```csharp
// Session data stored during login:
- CurrentUser (User ID)
- UserName
- UserRole
- UserEmail
- LoginTime
- HourlyRate (for lecturers)
- Department
- Claim counts
```

### Authorization Levels
- **Public**: Login page only
- **Lecturer**: Dashboard, Create Claim, View Own Claims
- **Programme Coordinator**: Approve Claims (First Level), View Pending
- **Academic Manager**: Final Approval, Reports
- **HR**: Full System Access, User Management, Reports

## 💻 Technology Stack

- **Framework**: ASP.NET Core 6.0 MVC
- **Database**: SQL Server with Entity Framework Core
- **Authentication**: Cookie-based Authentication
- **Session Management**: ASP.NET Core Session
- **Frontend**: Bootstrap 5, Font Awesome, JavaScript
- **Validation**: Data Annotations, jQuery Validation

## 📦 Installation & Setup

### Prerequisites
- .NET 6.0 SDK or later
- SQL Server 2019 or later
- Visual Studio 2022 or VS Code

### Steps

1. **Clone the Repository**
   ```bash
   git clone https://github.com/yourusername/ContractMonthlyClaimSystem.git
   cd ContractMonthlyClaimSystem
   ```

2. **Update Connection String**
   
   Edit `appsettings.json`:
   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ContractClaimSystemDB;Trusted_Connection=True;MultipleActiveResultSets=true"
     }
   }
   ```

3. **Run Database Migrations**
   ```bash
   dotnet ef database update
   ```
   
   Or the database will auto-create on first run.

4. **Run the Application**
   ```bash
   dotnet run
   ```

5. **Access the Application**
   
   Navigate to: `https://localhost:7xxx` (check console output)

## 👥 Default Users (Seeded)

| Username | Password | Role |
|----------|----------|------|
| hruser | hrpassword | HR |
| coordinator | coordpassword | Programme Coordinator |
| manager | managerpass | Academic Manager |
| hdabaAc | password123 | Lecturer |

**⚠️ Important**: Change these passwords immediately after first login in production!

## 🔑 Key Features Demonstration

### 1. HR Creates User
```csharp
// HR sets hourly rate during user creation
var user = new User
{
    Username = "newlecturer",
    Name = "John Doe",
    Email = "john@university.edu",
    Role = UserRole.Lecturer,
    HourlyRate = 150.00m,  // HR sets this
    Status = UserStatus.Active
};
```

### 2. Lecturer Submits Claim
```csharp
// Auto-calculation in view
function calculateTotal() {
    const hours = parseFloat(document.getElementById('hoursWorked').value);
    const rate = parseFloat(document.getElementById('hourlyRate').value);  // HR-set
    const total = hours * rate;
    document.getElementById('totalAmount').value = total.toFixed(2);
}
```

### 3. 180-Hour Validation
```csharp
// Server-side validation
if (model.HoursWorked > 180)
{
    ModelState.AddModelError("HoursWorked", 
        "Hours worked cannot exceed 180 hours per month.");
    return View(model);
}
```

### 4. Session Validation
```csharp
// Every protected action checks session
var sessionUser = HttpContext.Session.GetString("CurrentUser");
if (string.IsNullOrEmpty(sessionUser) || sessionUser != userId.ToString())
{
    return RedirectToAction("Login", "Account");
}
```

### 5. LINQ Report Generation
```csharp
// Example: Department summary using LINQ
var departmentSummary = claims
    .GroupBy(c => c.Lecturer.Department)
    .Select(g => new
    {
        Department = g.Key,
        ClaimCount = g.Count(),
        TotalAmount = g.Sum(c => c.TotalAmount),
        ApprovedAmount = g.Where(c => c.Status == ClaimStatus.Approved)
                          .Sum(c => c.TotalAmount)
    })
    .OrderByDescending(d => d.TotalAmount)
    .ToList();
```

## 📊 Database Schema

### Users Table
- UserId (PK)
- Username
- Name
- Email
- PasswordHash
- Role (Enum: Lecturer, ProgrammeCoordinator, AcademicManager, HR)
- HourlyRate ← **Set by HR**
- Department
- Status (Active/Inactive)
- CreatedDate
- LastLoginDate

### Claims Table
- ClaimId (PK)
- LecturerId (FK → Users)
- ClaimPeriod
- HoursWorked ← **Max 180 hours**
- HourlyRate ← **From User.HourlyRate**
- TotalAmount (Computed: HoursWorked × HourlyRate)
- Description
- Status (Draft, Submitted, UnderReview, Approved, Rejected)
- SubmissionDate
- FileData (Supporting document)

### Approvals Table
- ApprovalId (PK)
- ClaimId (FK → Claims)
- ApproverId (FK → Users)
- Status (Pending, Approved, Rejected)
- ApprovalDate
- Comments

## 🔄 Claim Workflow

```
1. Lecturer submits claim
   ↓
2. Programme Coordinator reviews
   ↓ (Approved)
3. Academic Manager reviews
   ↓ (Approved)
4. Claim marked as Approved
   ↓
5. HR can generate reports/invoices
```

## ✅ PROG6212 Checklist Compliance

### HR View
- [x] HR added as "super user" role
- [x] HR adds all users with complete information
- [x] HR can update all user information
- [x] HR can generate reports using LINQ
- [x] Reports downloadable (HTML format)
- [x] No public registration - HR creates all users

### Lecturer View
- [x] Login implemented
- [x] Hourly rate pulled from HR-set values
- [x] Auto-calculation implemented (Hours × Rate)
- [x] Validation for 180-hour maximum
- [x] Entity Framework with database
- [x] Track claims through approval process

### Admin Views (Separate)
- [x] Programme Coordinator view
- [x] Academic Manager view
- [x] Custom login implemented
- [x] **Sessions implemented throughout**
- [x] **Authorization prevents unauthorized access**

### Version Control
- [x] Regular commits to GitHub (10+ commits)
- [x] Descriptive commit messages

### Documentation
- [x] README updated with Part 2 to Part 3 changes
- [x] Key features documented
- [x] Installation instructions included

## 🎥 Video Demonstration

[YouTube Link - Unlisted]

**Video Contents**:
1. HR creating new users (0:00-2:00)
2. HR setting hourly rates (2:00-3:00)
3. Lecturer login and claim submission (3:00-5:00)
4. Auto-calculation demonstration (5:00-6:00)
5. 180-hour validation (6:00-7:00)
6. Programme Coordinator approval (7:00-9:00)
7. Academic Manager final approval (9:00-11:00)
8. Session functionality demo (11:00-12:00)
9. Report generation (12:00-14:00)
10. Unauthorized access prevention (14:00-15:00)

## 🐛 Known Issues & Future Enhancements

### Current Limitations
- PDF generation uses HTML format (can be enhanced with iTextSharp)
- No email notifications (can be added with SMTP)
- Basic file upload validation

### Planned Enhancements
- Email notifications for approvals
- PDF generation with proper library
- Advanced reporting with charts
- Bulk user import from Excel
- Export claims to Excel

## 📝 Testing

### Test Scenarios
1. **HR Functions**
   - Create user with all fields
   - Edit existing user
   - Activate/deactivate users
   - Reset passwords

2. **Lecturer Functions**
   - Submit claim with auto-calculation
   - Test 180-hour validation
   - Upload supporting document
   - View claim status

3. **Approval Workflow**
   - Coordinator approval
   - Manager final approval
   - Claim rejection with comments

4. **Security**
   - Unauthorized access attempts
   - Session expiration
   - Role-based restrictions

5. **Reports**
   - Monthly report generation
   - Lecturer performance report
   - LINQ query verification

## 🤝 Contributing

This is an academic project for PROG6212. For educational purposes only.

## 📄 License

This project is created for academic purposes as part of PROG6212 coursework.

## 👨‍💻 Author

**Your Name**
- Student Number: XXXXXXXXX
- Institution: Independent Institute of Education
- Module: PROG6212
- Year: 2024

## 📞 Support

For issues or questions:
1. Check the documentation
2. Review the video demonstration
3. Contact: your.email@student.iie.ac.za

---

## 🎓 Academic Integrity Statement

This project represents original work completed for PROG6212. All code and documentation have been created by the author unless otherwise cited.

**Last Updated**: November 2024
**Version**: 3.0 (Final POE)
