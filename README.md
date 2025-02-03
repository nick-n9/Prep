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

