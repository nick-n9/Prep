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

-----------------------------------------------------------


-------------------------------------------------------------




Blog Title: Behind the Scenes of a Scalable Recruitment Database Design
Introduction:
Hey! If you’ve been following my work lately, you know I’ve been putting together a recruitment management system, and today I wanted to share a deep dive into the database design. Designing a robust and scalable database is the foundation of any application, and it’s no different here. In this post, I’ll walk through the key decisions I made while designing the database schema, the relationships between the tables, and how each element is optimized to ensure the system can scale as the user base and data grow.

1. The Core of the Database Design: Normalization and Efficiency
One of my primary goals when designing the database was to keep it normalized. This prevents data redundancy and maintains integrity, making the database easier to maintain as it grows. Let’s go over the key tables I built and the rationale behind each decision.

Job Creation & Position Management
The Positions table holds the core details of each job opening. Each job needs to track its title, description, and required skills. I included a status field to track whether the job is open, closed, or on hold. This status is important because recruiters can update the position’s state, and the database should reflect that dynamically.

Why this helps scale: The positions table could grow rapidly with job postings. By keeping it separate, we avoid bloating other tables (like candidate profiles) with unnecessary position data. The Skills table is also separate, with a many-to-many relationship to positions, which makes it easy to add or modify required skills without altering other data.
Candidate Profile Management
The Candidates table stores the core details about each candidate—like their name, email, resume, and contact information. A key decision here was to use a separate table for candidate skills to track the skills each candidate possesses. The CandidateSkills table links candidates and skills in a many-to-many relationship.

Why this helps scale: By separating candidate profiles and skills, we ensure that we don’t repeatedly store skill information in the candidate table. This becomes crucial as the number of candidates grows. Instead, we link candidates to skills via the CandidateSkills table, which maintains flexibility and keeps queries efficient.
Resume Screening & Shortlisting
For the resume screening process, the CandidateReviews table allows recruiters and reviewers to leave feedback on candidate profiles. They can rate skills and leave comments. This is linked to the Candidates table via CandidateID. We also track the status of the candidate (e.g., screened, shortlisted, rejected).

Why this helps scale: This allows multiple reviewers to leave comments without overcrowding the candidate table itself. It also ensures that we can track each review separately, making it easier to scale when more candidates need to be reviewed. The status column helps keep track of where the candidate is in the recruitment process, simplifying reporting.
Interview Scheduling & Feedback
The Interviews table stores each interview's details (date, type, feedback). Each interview can have multiple InterviewRounds, which is why I separated that into its own table. This means a candidate can go through multiple rounds (e.g., technical and HR), with feedback added for each round.

Why this helps scale: By keeping interviews and rounds separate, we avoid storing redundant data. If a candidate goes through multiple rounds, it doesn’t inflate the interview record. Instead, it allows easy extension to track more interview rounds without affecting the overall structure. Interview feedback can also be linked to each round, making reporting much easier.
Final Selection & Document Verification
Once a candidate is selected, their profile gets moved to the EmployeeRecords table. The OfferLetters table holds details of the offer, and the Documents table tracks uploaded candidate documents. Verification status (whether background checks, references, etc., are completed) is also part of the flow.

Why this helps scale: The EmployeeRecords table becomes crucial once candidates are converted to employees. By separating offer letters and documents into their own tables, I prevent cluttering the main candidate table. This makes it easier to scale up when handling larger datasets without performance hits.
2. Relationships & How They Tie Everything Together
Now, let’s talk about relationships and how they were designed to make sure everything ties together efficiently and scales.

One-to-Many Relationships
One Position to Many Candidates: A position can have many candidates applying to it, but each candidate can only apply for one position at a time. This is a one-to-many relationship between Positions and Candidates.
One Candidate to Many Interviews: Each candidate may go through multiple interviews, so this is another one-to-many relationship, linking Candidates to Interviews.
One Recruiter to Many Reviews: A recruiter may review many candidates, so there’s a one-to-many relationship between the Recruiters and CandidateReviews.
Many-to-Many Relationships
Candidates and Skills: A candidate can have multiple skills, and each skill can be relevant to many candidates. This was handled with a many-to-many relationship between Candidates and Skills, with a linking table called CandidateSkills.
Positions and Skills: A position can require multiple skills, and each skill can be relevant to many positions. Again, I used a many-to-many relationship with a linking table called PositionSkills.
Why Relationships Are Crucial for Scalability
The relationships were carefully structured to ensure the database can grow efficiently. For instance:

Many-to-many relationships are especially important in large-scale applications. Instead of duplicating data across tables, linking tables like CandidateSkills and PositionSkills allow me to store and update data in one place, keeping everything flexible.
By separating interviews into different rounds, I can easily extend the application to add more interview stages without refactoring the entire schema. Interviewers can give feedback at different stages, and I can easily query feedback per round for reporting.
3. Indexing for Performance and Scalability
As the application grows, so will the amount of data. To keep things fast, I focused heavily on indexing. By indexing fields like CandidateID, PositionID, and Status, I’m ensuring that common queries (like “list all candidates for a position” or “get all candidates in a specific status”) will remain fast even with large datasets.

Why this helps scale: Without indexing, every query would require scanning the entire table, which can be super slow as data grows. Proper indexing ensures quick retrieval of data, even in large tables.
4. Data Integrity and Transactions
With multiple users accessing and modifying the data, ensuring data integrity is critical. I made sure to include proper transaction management where necessary. For example, when updating a candidate's status or adding feedback, the changes happen within a single transaction, so if anything goes wrong, it gets rolled back.

Why this helps scale: As more users interact with the database, ensuring that data stays consistent even when multiple users are editing records is key to maintaining a stable system. Transactions make sure nothing gets left in an inconsistent state.
5. The Bottom Line: Why This Design Scales
To wrap it all up, the database was designed with the future in mind. Here’s why it scales:

Normalization ensures data consistency and minimizes redundancy, which becomes crucial as the system grows.
Efficient relationships (like many-to-many and one-to-many) ensure the database can expand without performance hits.
Indexes and partitioning keep the system responsive as the data grows.
Transaction management and error handling ensure that the database stays consistent even as many users interact with it simultaneously.
------------------------------------------------

------------------------------------------------
