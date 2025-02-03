public class Position
{
    public int PositionID { get; set; }
    public string JobTitle { get; set; }
    public string JobDescription { get; set; }
    public string MinRequiredSkills { get; set; }
    public string PreferredSkills { get; set; }
    public string Status { get; set; }
    public string ReasonForClosure { get; set; }
    public DateTime? PositionClosedDate { get; set; }
    public int? LinkedCandidateID { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime? ModifiedDate { get; set; }

    // Navigation property
    public Candidate LinkedCandidate { get; set; }
    public ICollection<PositionSkill> PositionSkills { get; set; }
    public ICollection<CandidateReview> CandidateReviews { get; set; }
    public ICollection<Interview> Interviews { get; set; }
    public ICollection<FinalSelection> FinalSelections { get; set; }
}

public class Skill
{
    public int SkillID { get; set; }
    public string SkillName { get; set; }

    // Navigation property
    public ICollection<PositionSkill> PositionSkills { get; set; }
    public ICollection<CandidateSkill> CandidateSkills { get; set; }
}

public class PositionSkill
{
    public int PositionSkillID { get; set; }
    public int PositionID { get; set; }
    public int SkillID { get; set; }

    // Navigation properties
    public Position Position { get; set; }
    public Skill Skill { get; set; }
}

public class Candidate
{
    public int CandidateID { get; set; }
    public string FullName { get; set; }
    public string Email { get; set; }
    public string PhoneNumber { get; set; }
    public string ResumePath { get; set; }
    public string ProfileStatus { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime? ModifiedDate { get; set; }

    // Navigation properties
    public ICollection<CandidateSkill> CandidateSkills { get; set; }
    public ICollection<CandidateDocument> CandidateDocuments { get; set; }
    public ICollection<CandidateReview> CandidateReviews { get; set; }
    public ICollection<Interview> Interviews { get; set; }
    public ICollection<FinalSelection> FinalSelections { get; set; }
    public ICollection<CandidateStatusHistory> CandidateStatusHistories { get; set; }
    public ICollection<DocumentVerification> DocumentVerifications { get; set; }
}

public class CandidateSkill
{
    public int CandidateSkillID { get; set; }
    public int CandidateID { get; set; }
    public int SkillID { get; set; }
    public int YearsOfExperience { get; set; }

    // Navigation properties
    public Candidate Candidate { get; set; }
    public Skill Skill { get; set; }
}

public class CandidateDocument
{
    public int DocumentID { get; set; }
    public int CandidateID { get; set; }
    public string DocumentType { get; set; }
    public string DocumentPath { get; set; }
    public string VerificationStatus { get; set; }
    public DateTime CreatedDate { get; set; }

    // Navigation property
    public Candidate Candidate { get; set; }
}

public class CandidateReview
{
    public int ReviewID { get; set; }
    public int CandidateID { get; set; }
    public int PositionID { get; set; }
    public string ReviewerID { get; set; }  // IdentityUser ID (string)
    public DateTime ReviewDate { get; set; }
    public string Comments { get; set; }
    public string Status { get; set; }

    // Navigation properties
    public Candidate Candidate { get; set; }
    public Position Position { get; set; }
    public IdentityUser Reviewer { get; set; }  // Navigation to AspNetUser
}

public class Interview
{
    public int InterviewID { get; set; }
    public int CandidateID { get; set; }
    public int PositionID { get; set; }
    public string InterviewType { get; set; }
    public DateTime InterviewDate { get; set; }
    public string InterviewerID { get; set; }  // IdentityUser ID (string)
    public string Status { get; set; }
    public string Feedback { get; set; }

    // Navigation properties
    public Candidate Candidate { get; set; }
    public Position Position { get; set; }
    public IdentityUser Interviewer { get; set; }  // Navigation to AspNetUser
}

public class InterviewRound
{
    public int InterviewRoundID { get; set; }
    public int InterviewID { get; set; }
    public int RoundNumber { get; set; }
    public string RoundType { get; set; }
    public string Feedback { get; set; }
    public int Rating { get; set; }

    // Navigation properties
    public Interview Interview { get; set; }
}

public class InterviewFeedback
{
    public int FeedbackID { get; set; }
    public int InterviewRoundID { get; set; }
    public string InterviewerID { get; set; }  // IdentityUser ID (string)
    public string FeedbackText { get; set; }
    public int Rating { get; set; }
    public DateTime CreatedDate { get; set; }

    // Navigation properties
    public InterviewRound InterviewRound { get; set; }
    public IdentityUser Interviewer { get; set; }  // Navigation to AspNetUser
}

public class DocumentVerification
{
    public int VerificationID { get; set; }
    public int CandidateID { get; set; }
    public string VerificationStatus { get; set; }
    public DateTime VerificationDate { get; set; }
    public string VerifiedBy { get; set; }  // IdentityUser ID (string)

    // Navigation properties
    public Candidate Candidate { get; set; }
    public IdentityUser VerifiedByUser { get; set; }  // Navigation to AspNetUser
}

public class FinalSelection
{
    public int SelectionID { get; set; }
    public int CandidateID { get; set; }
    public int PositionID { get; set; }
    public string OfferLetterPath { get; set; }
    public DateTime JoiningDate { get; set; }
    public string Status { get; set; }

    // Navigation properties
    public Candidate Candidate { get; set; }
    public Position Position { get; set; }
}

public class CandidateStatusHistory
{
    public int StatusHistoryID { get; set; }
    public int CandidateID { get; set; }
    public string Status { get; set; }
    public string StatusReason { get; set; }
    public string ChangedBy { get; set; }  // IdentityUser ID (string)
    public DateTime ChangedDate { get; set; }

    // Navigation properties
    public Candidate Candidate { get; set; }
    public IdentityUser ChangedByUser { get; set; }  // Navigation to AspNetUser
}

public class Notification
{
    public int NotificationID { get; set; }
    public string UserID { get; set; }  // IdentityUser ID (string)
    public string Message { get; set; }
    public bool IsRead { get; set; }
    public DateTime CreatedDate { get; set; }

    // Navigation property
    public IdentityUser User { get; set; }  // Navigation to AspNetUser
}
--------------------------------------------------------------

--------------------------------------------------------------

using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;
using YourNamespace.Models;

public class ApplicationDbContext : IdentityDbContext<IdentityUser>
{
    public DbSet<Position> Positions { get; set; }
    public DbSet<Skill> Skills { get; set; }
    public DbSet<PositionSkill> PositionSkills { get; set; }
    public DbSet<Candidate> Candidates { get; set; }
    public DbSet<CandidateSkill> CandidateSkills { get; set; }
    public DbSet<CandidateDocument> CandidateDocuments { get; set; }
    public DbSet<CandidateReview> CandidateReviews { get; set; }
    public DbSet<Interview> Interviews { get; set; }
    public DbSet<InterviewRound> InterviewRounds { get; set; }
    public DbSet<InterviewFeedback> InterviewFeedbacks { get; set; }
    public DbSet<DocumentVerification> DocumentVerifications { get; set; }
    public DbSet<FinalSelection> FinalSelections { get; set; }
    public DbSet<CandidateStatusHistory> CandidateStatusHistories { get; set; }
    public DbSet<Notification> Notifications { get; set; }

    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Defining relationships explicitly for Identity references (IdentityUser)
        modelBuilder.Entity<CandidateReview>()
            .HasOne(cr => cr.Reviewer)
            .WithMany()
            .HasForeignKey(cr => cr.ReviewerID);

        modelBuilder.Entity<Interview>()
            .HasOne(i => i.Interviewer)
            .WithMany()
            .HasForeignKey(i => i.InterviewerID);

        modelBuilder.Entity<InterviewFeedback>()
            .HasOne(ifb => ifb.Interviewer)
            .WithMany()
            .HasForeignKey(ifb => ifb.InterviewerID);

        modelBuilder.Entity<DocumentVerification>()
            .HasOne(dv => dv.VerifiedByUser)
            .WithMany()
            .HasForeignKey(dv => dv.VerifiedBy);

        modelBuilder.Entity<CandidateStatusHistory>()
            .HasOne(csh => csh.ChangedByUser)
            .WithMany()
            .HasForeignKey(csh => csh.ChangedBy);

        modelBuilder.Entity<Notification>()
            .HasOne(n => n.User)
            .WithMany()
            .HasForeignKey(n => n.UserID);

        // Ensure the AspNetUser ID for Identity is treated as a string
        modelBuilder.Entity<IdentityUser>().Property(u => u.Id).HasColumnType("nvarchar(450)");
    }
}
---------------------------------------------------------------------
--------------------------------------------------------------------
---------------------------------------------------------------






























{
  "orderDate": "2025-02-03T10:00:00",
  "deliveryDate": "2025-02-10T10:00:00",
  "status": "PENDING",
  "customer": {
    "id": 1
  },
  "items": [
    {
      "id": 2
    },
    {
      "id": 3
    }
  ]
}
----------------------------------
{
  "id": 5,
  "orderDate": "2025-02-03T10:00:00",
  "deliveryDate": "2025-02-10T10:00:00",
  "status": "PENDING",
  "customer": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "items": [
    {
      "id": 2,
      "name": "Laptop",
      "price": 800
    },
    {
      "id": 3,
      "name": "Mouse",
      "price": 20
    }
  ]
}
--------------------------
5. Cancel an Order
Endpoint: PUT /orders/{orderId}/cancel
Example: PUT /orders/5/cancel
Expected Response (200 OK):
{
  "message": "Order 5 has been cancelled successfully"
}
-------------------------

-- .NET Core Identity Tables
CREATE TABLE AspNetUsers (
    Id NVARCHAR(450) PRIMARY KEY,
    UserName NVARCHAR(256) NOT NULL,
    NormalizedUserName NVARCHAR(256) NOT NULL,
    Email NVARCHAR(256) NULL,
    NormalizedEmail NVARCHAR(256) NULL,
    EmailConfirmed BIT NOT NULL DEFAULT 0,
    PasswordHash NVARCHAR(MAX) NULL,
    SecurityStamp NVARCHAR(MAX) NULL,
    ConcurrencyStamp NVARCHAR(MAX) NULL,
    PhoneNumber NVARCHAR(20) NULL,
    PhoneNumberConfirmed BIT NOT NULL DEFAULT 0,
    TwoFactorEnabled BIT NOT NULL DEFAULT 0,
    LockoutEnd DATETIMEOFFSET(7) NULL,
    LockoutEnabled BIT NOT NULL DEFAULT 0,
    AccessFailedCount INT NOT NULL DEFAULT 0,
    CreatedAt DATETIME DEFAULT GETDATE(),
    LastLoginDate DATETIME NULL
);

CREATE TABLE AspNetRoles (
    Id NVARCHAR(450) PRIMARY KEY,
    Name NVARCHAR(256) NULL,
    NormalizedName NVARCHAR(256) NULL,
    ConcurrencyStamp NVARCHAR(MAX) NULL
);

CREATE TABLE AspNetUserRoles (
    UserId NVARCHAR(450) NOT NULL,
    RoleId NVARCHAR(450) NOT NULL,
    PRIMARY KEY (UserId, RoleId),
    FOREIGN KEY (UserId) REFERENCES AspNetUsers(Id) ON DELETE CASCADE,
    FOREIGN KEY (RoleId) REFERENCES AspNetRoles(Id) ON DELETE CASCADE
);

CREATE TABLE AspNetUserClaims (
    Id INT PRIMARY KEY IDENTITY,
    UserId NVARCHAR(450) NOT NULL,
    ClaimType NVARCHAR(MAX) NULL,
    ClaimValue NVARCHAR(MAX) NULL,
    FOREIGN KEY (UserId) REFERENCES AspNetUsers(Id) ON DELETE CASCADE
);

CREATE TABLE AspNetRoleClaims (
    Id INT PRIMARY KEY IDENTITY,
    RoleId NVARCHAR(450) NOT NULL,
    ClaimType NVARCHAR(MAX) NULL,
    ClaimValue NVARCHAR(MAX) NULL,
    FOREIGN KEY (RoleId) REFERENCES AspNetRoles(Id) ON DELETE CASCADE
);

CREATE TABLE AspNetUserLogins (
    LoginProvider NVARCHAR(128) NOT NULL,
    ProviderKey NVARCHAR(128) NOT NULL,
    ProviderDisplayName NVARCHAR(256) NULL,
    UserId NVARCHAR(450) NOT NULL,
    PRIMARY KEY (LoginProvider, ProviderKey),
    FOREIGN KEY (UserId) REFERENCES AspNetUsers(Id) ON DELETE CASCADE
);

CREATE TABLE AspNetUserTokens (
    UserId NVARCHAR(450) NOT NULL,
    LoginProvider NVARCHAR(128) NOT NULL,
    Name NVARCHAR(128) NOT NULL,
    Value NVARCHAR(MAX) NULL,
    PRIMARY KEY (UserId, LoginProvider, Name),
    FOREIGN KEY (UserId) REFERENCES AspNetUsers(Id) ON DELETE CASCADE
);

-- Recruitment Tables

-- Positions Table
CREATE TABLE Positions (
    PositionID INT PRIMARY KEY IDENTITY,
    JobTitle VARCHAR(255) NOT NULL,
    JobDescription TEXT,
    MinRequiredSkills TEXT,
    PreferredSkills TEXT,
    Status VARCHAR(50),
    ReasonForClosure TEXT,
    PositionClosedDate DATETIME,
    LinkedCandidateID INT NULL,
    CreatedDate DATETIME DEFAULT GETDATE(),
    ModifiedDate DATETIME
);

-- Skills Table
CREATE TABLE Skills (
    SkillID INT PRIMARY KEY IDENTITY,
    SkillName VARCHAR(255) NOT NULL
);

-- PositionSkills Table
CREATE TABLE PositionSkills (
    PositionSkillID INT PRIMARY KEY IDENTITY,
    PositionID INT,
    SkillID INT,
    FOREIGN KEY (PositionID) REFERENCES Positions(PositionID),
    FOREIGN KEY (SkillID) REFERENCES Skills(SkillID)
);

-- Candidates Table
CREATE TABLE Candidates (
    CandidateID INT PRIMARY KEY IDENTITY,
    FullName VARCHAR(255),
    Email VARCHAR(255) UNIQUE,
    PhoneNumber VARCHAR(50),
    ResumePath VARCHAR(255),
    ProfileStatus VARCHAR(50),
    CreatedDate DATETIME DEFAULT GETDATE(),
    ModifiedDate DATETIME
);

-- CandidateSkills Table
CREATE TABLE CandidateSkills (
    CandidateSkillID INT PRIMARY KEY IDENTITY,
    CandidateID INT,
    SkillID INT,
    YearsOfExperience INT,
    FOREIGN KEY (CandidateID) REFERENCES Candidates(CandidateID),
    FOREIGN KEY (SkillID) REFERENCES Skills(SkillID)
);

-- CandidateDocuments Table
CREATE TABLE CandidateDocuments (
    DocumentID INT PRIMARY KEY IDENTITY,
    CandidateID INT,
    DocumentType VARCHAR(50),
    DocumentPath VARCHAR(255),
    VerificationStatus VARCHAR(50),
    CreatedDate DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (CandidateID) REFERENCES Candidates(CandidateID)
);

-- CandidateReviews Table
CREATE TABLE CandidateReviews (
    ReviewID INT PRIMARY KEY IDENTITY,
    CandidateID INT,
    PositionID INT,
    ReviewerID INT,
    ReviewDate DATETIME DEFAULT GETDATE(),
    Comments TEXT,
    Status VARCHAR(50),
    FOREIGN KEY (CandidateID) REFERENCES Candidates(CandidateID),
    FOREIGN KEY (PositionID) REFERENCES Positions(PositionID),
    FOREIGN KEY (ReviewerID) REFERENCES AspNetUsers(Id)
);

-- Interviews Table
CREATE TABLE Interviews (
    InterviewID INT PRIMARY KEY IDENTITY,
    CandidateID INT,
    PositionID INT,
    InterviewType VARCHAR(50),
    InterviewDate DATETIME,
    InterviewerID INT,
    Status VARCHAR(50),
    Feedback TEXT,
    FOREIGN KEY (CandidateID) REFERENCES Candidates(CandidateID),
    FOREIGN KEY (PositionID) REFERENCES Positions(PositionID),
    FOREIGN KEY (InterviewerID) REFERENCES AspNetUsers(Id)
);

-- InterviewRounds Table
CREATE TABLE InterviewRounds (
    InterviewRoundID INT PRIMARY KEY IDENTITY,
    InterviewID INT,
    RoundNumber INT,
    RoundType VARCHAR(50),
    Feedback TEXT,
    Rating INT,
    FOREIGN KEY (InterviewID) REFERENCES Interviews(InterviewID)
);

-- InterviewFeedback Table
CREATE TABLE InterviewFeedback (
    FeedbackID INT PRIMARY KEY IDENTITY,
    InterviewRoundID INT,
    InterviewerID INT,
    FeedbackText TEXT,
    Rating INT,
    CreatedDate DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (InterviewRoundID) REFERENCES InterviewRounds(InterviewRoundID),
    FOREIGN KEY (InterviewerID) REFERENCES AspNetUsers(Id)
);

-- DocumentVerification Table
CREATE TABLE DocumentVerification (
    VerificationID INT PRIMARY KEY IDENTITY,
    CandidateID INT,
    VerificationStatus VARCHAR(50),
    VerificationDate DATETIME,
    VerifiedBy INT,
    FOREIGN KEY (CandidateID) REFERENCES Candidates(CandidateID),
    FOREIGN KEY (VerifiedBy) REFERENCES AspNetUsers(Id)
);

-- FinalSelection Table
CREATE TABLE FinalSelection (
    SelectionID INT PRIMARY KEY IDENTITY,
    CandidateID INT,
    PositionID INT,
    OfferLetterPath VARCHAR(255),
    JoiningDate DATETIME,
    Status VARCHAR(50),
    FOREIGN KEY (CandidateID) REFERENCES Candidates(CandidateID),
    FOREIGN KEY (PositionID) REFERENCES Positions(PositionID)
);

-- CandidateStatusHistory Table
CREATE TABLE CandidateStatusHistory (
    StatusHistoryID INT PRIMARY KEY IDENTITY,
    CandidateID INT,
    Status VARCHAR(50),
    StatusReason TEXT,
    ChangedBy INT,
    ChangedDate DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (CandidateID) REFERENCES Candidates(CandidateID),
    FOREIGN KEY (ChangedBy) REFERENCES AspNetUsers(Id)
);

-- Notifications Table
CREATE TABLE Notifications (
    NotificationID INT PRIMARY KEY IDENTITY,
    UserID INT,
    Message TEXT,
    IsRead BIT DEFAULT 0,
    CreatedDate DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (UserID) REFERENCES AspNetUsers(Id)
);

