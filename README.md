Install Qt and SQLite: Before you begin, you need to install the Qt framework and SQLite. You can install Qt from here and SQLite can be added via Qt’s SQL module.

Include Necessary Headers: #include <QtCore>
#include <QtGui>
#include <QtWidgets>
#include <QtSql>
Database Setup (SQLite for managing user and content data):
class DatabaseManager {
public:
    QSqlDatabase db;

    DatabaseManager() {
        db = QSqlDatabase::addDatabase("QSQLITE");
        db.setDatabaseName("social_media_db.sqlite");

        if (!db.open()) {
            qWarning() << "Error: Unable to open database!";
        }

        createTables();
    }

    void createTables() {
        QSqlQuery query;
        
        // Creating a table for users
        query.exec("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, username TEXT, email TEXT, role TEXT)");

        // Creating a table for content
        query.exec("CREATE TABLE IF NOT EXISTS content (id INTEGER PRIMARY KEY, user_id INTEGER, content TEXT, date TEXT)");

        // Creating a table for analytics
        query.exec("CREATE TABLE IF NOT EXISTS analytics (id INTEGER PRIMARY KEY, user_id INTEGER, actions TEXT, timestamp TEXT)");
    }

    // Function to add a new user
    void addUser(const QString &username, const QString &email, const QString &role) {
        QSqlQuery query;
        query.prepare("INSERT INTO users (username, email, role) VALUES (?, ?, ?)");
        query.addBindValue(username);
        query.addBindValue(email);
        query.addBindValue(role);
        query.exec();
    }

    // Function to add content
    void addContent(int userId, const QString &content) {
        QSqlQuery query;
        query.prepare("INSERT INTO content (user_id, content, date) VALUES (?, ?, ?)");
        query.addBindValue(userId);
        query.addBindValue(content);
        query.addBindValue(QDateTime::currentDateTime().toString());
        query.exec();
    }

    // Function to get user data
    QSqlQuery getUserData() {
        QSqlQuery query("SELECT * FROM users");
        return query;
    }

    // Function to get content data
    QSqlQuery getContentData() {
        QSqlQuery query("SELECT * FROM content");
        return query;
    }

    // Function to get analytics data
    QSqlQuery getAnalyticsData() {
        QSqlQuery query("SELECT * FROM analytics");
        return query;
    }
};
Admin Dashboard UI using Qt:
You would create a basic interface with sections like:

User Management: Add, remove, update users.
Content Moderation: View content and delete inappropriate posts.
Analytics: View user actions, active users, etc.
class AdminDashboard : public QWidget {
    Q_OBJECT
public:
    AdminDashboard() {
        setupUI();
        dbManager = new DatabaseManager();
    }

    ~AdminDashboard() {
        delete dbManager;
    }

private:
    DatabaseManager *dbManager;
    QTableWidget *userTable;
    QTableWidget *contentTable;
    QTableWidget *analyticsTable;

    void setupUI() {
        QVBoxLayout *layout = new QVBoxLayout(this);

        // User Management Section
        userTable = new QTableWidget(this);
        layout->addWidget(new QLabel("User Management"));
        layout->addWidget(userTable);

        QPushButton *addUserBtn = new QPushButton("Add User", this);
        layout->addWidget(addUserBtn);

        connect(addUserBtn, &QPushButton::clicked, this, &AdminDashboard::addUser);

        // Content Moderation Section
        contentTable = new QTableWidget(this);
        layout->addWidget(new QLabel("Content Moderation"));
        layout->addWidget(contentTable);

        QPushButton *deleteContentBtn = new QPushButton("Delete Content", this);
        layout->addWidget(deleteContentBtn);
        connect(deleteContentBtn, &QPushButton::clicked, this, &AdminDashboard::deleteContent);

        // Analytics Section
        analyticsTable = new QTableWidget(this);
        layout->addWidget(new QLabel("Analytics"));
        layout->addWidget(analyticsTable);

        QPushButton *refreshAnalyticsBtn = new QPushButton("Refresh Analytics", this);
        layout->addWidget(refreshAnalyticsBtn);
        connect(refreshAnalyticsBtn, &QPushButton::clicked, this, &AdminDashboard::refreshAnalytics);

        setLayout(layout);
    }

    // Function to add a user
    void addUser() {
        QString username = "User" + QString::number(rand() % 1000);
        dbManager->addUser(username, username + "@example.com", "admin");
        loadUserData();
    }

    // Function to delete content
    void deleteContent() {
        dbManager->getContentData();
        // For simplicity, assume we delete the first content
        QSqlQuery query("DELETE FROM content WHERE id=1");
        query.exec();
        loadContentData();
    }

    // Function to refresh analytics
    void refreshAnalytics() {
        loadAnalyticsData();
    }

    // Function to load user data into the UI
    void loadUserData() {
        QSqlQuery query = dbManager->getUserData();
        userTable->setRowCount(0);
        while (query.next()) {
            int row = userTable->rowCount();
            userTable->insertRow(row);
            userTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            userTable->setItem(row, 1, new QTableWidgetItem(query.value("username").toString()));
            userTable->setItem(row, 2, new QTableWidgetItem(query.value("email").toString()));
            userTable->setItem(row, 3, new QTableWidgetItem(query.value("role").toString()));
        }
    }

    // Function to load content data into the UI
    void loadContentData() {
        QSqlQuery query = dbManager->getContentData();
        contentTable->setRowCount(0);
        while (query.next()) {
            int row = contentTable->rowCount();
            contentTable->insertRow(row);
            contentTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            contentTable->setItem(row, 1, new QTableWidgetItem(query.value("user_id").toString()));
            contentTable->setItem(row, 2, new QTableWidgetItem(query.value("content").toString()));
            contentTable->setItem(row, 3, new QTableWidgetItem(query.value("date").toString()));
        }
    }

    // Function to load analytics data into the UI
    void loadAnalyticsData() {
        QSqlQuery query = dbManager->getAnalyticsData();
        analyticsTable->setRowCount(0);
        while (query.next()) {
            int row = analyticsTable->rowCount();
            analyticsTable->insertRow(row);
            analyticsTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            analyticsTable->setItem(row, 1, new QTableWidgetItem(query.value("user_id").toString()));
            analyticsTable->setItem(row, 2, new QTableWidgetItem(query.value("actions").toString()));
            analyticsTable->setItem(row, 3, new QTableWidgetItem(query.value("timestamp").toString()));
        }
    }
};
class AdminDashboard : public QWidget {
    Q_OBJECT
public:
    AdminDashboard() {
        setupUI();
        dbManager = new DatabaseManager();
    }

    ~AdminDashboard() {
        delete dbManager;
    }

private:
    DatabaseManager *dbManager;
    QTableWidget *userTable;
    QTableWidget *contentTable;
    QTableWidget *analyticsTable;

    void setupUI() {
        QVBoxLayout *layout = new QVBoxLayout(this);

        // User Management Section
        userTable = new QTableWidget(this);
        layout->addWidget(new QLabel("User Management"));
        layout->addWidget(userTable);

        QPushButton *addUserBtn = new QPushButton("Add User", this);
        layout->addWidget(addUserBtn);

        connect(addUserBtn, &QPushButton::clicked, this, &AdminDashboard::addUser);

        // Content Moderation Section
        contentTable = new QTableWidget(this);
        layout->addWidget(new QLabel("Content Moderation"));
        layout->addWidget(contentTable);

        QPushButton *deleteContentBtn = new QPushButton("Delete Content", this);
        layout->addWidget(deleteContentBtn);
        connect(deleteContentBtn, &QPushButton::clicked, this, &AdminDashboard::deleteContent);

        // Analytics Section
        analyticsTable = new QTableWidget(this);
        layout->addWidget(new QLabel("Analytics"));
        layout->addWidget(analyticsTable);

        QPushButton *refreshAnalyticsBtn = new QPushButton("Refresh Analytics", this);
        layout->addWidget(refreshAnalyticsBtn);
        connect(refreshAnalyticsBtn, &QPushButton::clicked, this, &AdminDashboard::refreshAnalytics);

        setLayout(layout);
    }

    // Function to add a user
    void addUser() {
        QString username = "User" + QString::number(rand() % 1000);
        dbManager->addUser(username, username + "@example.com", "admin");
        loadUserData();
    }

    // Function to delete content
    void deleteContent() {
        dbManager->getContentData();
        // For simplicity, assume we delete the first content
        QSqlQuery query("DELETE FROM content WHERE id=1");
        query.exec();
        loadContentData();
    }

    // Function to refresh analytics
    void refreshAnalytics() {
        loadAnalyticsData();
    }

    // Function to load user data into the UI
    void loadUserData() {
        QSqlQuery query = dbManager->getUserData();
        userTable->setRowCount(0);
        while (query.next()) {
            int row = userTable->rowCount();
            userTable->insertRow(row);
            userTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            userTable->setItem(row, 1, new QTableWidgetItem(query.value("username").toString()));
            userTable->setItem(row, 2, new QTableWidgetItem(query.value("email").toString()));
            userTable->setItem(row, 3, new QTableWidgetItem(query.value("role").toString()));
        }
    }

    // Function to load content data into the UI
    void loadContentData() {
        QSqlQuery query = dbManager->getContentData();
        contentTable->setRowCount(0);
        while (query.next()) {
            int row = contentTable->rowCount();
            contentTable->insertRow(row);
            contentTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            contentTable->setItem(row, 1, new QTableWidgetItem(query.value("user_id").toString()));
            contentTable->setItem(row, 2, new QTableWidgetItem(query.value("content").toString()));
            contentTable->setItem(row, 3, new QTableWidgetItem(query.value("date").toString()));
        }
    }

    // Function to load analytics data into the UI
    void loadAnalyticsData() {
        QSqlQuery query = dbManager->getAnalyticsData();
        analyticsTable->setRowCount(0);
        while (query.next()) {
            int row = analyticsTable->rowCount();
            analyticsTable->insertRow(row);
            analyticsTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            analyticsTable->setItem(row, 1, new QTableWidgetItem(query.value("user_id").toString()));
            analyticsTable->setItem(row, 2, new QTableWidgetItem(query.value("actions").toString()));
            analyticsTable->setItem(row, 3, new QTableWidgetItem(query.value("timestamp").toString()));
        }
    }
};
class AdminDashboard : public QWidget {
    Q_OBJECT
public:
    AdminDashboard() {
        setupUI();
        dbManager = new DatabaseManager();
    }

    ~AdminDashboard() {
        delete dbManager;
    }

private:
    DatabaseManager *dbManager;
    QTableWidget *userTable;
    QTableWidget *contentTable;
    QTableWidget *analyticsTable;

    void setupUI() {
        QVBoxLayout *layout = new QVBoxLayout(this);

        // User Management Section
        userTable = new QTableWidget(this);
        layout->addWidget(new QLabel("User Management"));
        layout->addWidget(userTable);

        QPushButton *addUserBtn = new QPushButton("Add User", this);
        layout->addWidget(addUserBtn);

        connect(addUserBtn, &QPushButton::clicked, this, &AdminDashboard::addUser);

        // Content Moderation Section
        contentTable = new QTableWidget(this);
        layout->addWidget(new QLabel("Content Moderation"));
        layout->addWidget(contentTable);

        QPushButton *deleteContentBtn = new QPushButton("Delete Content", this);
        layout->addWidget(deleteContentBtn);
        connect(deleteContentBtn, &QPushButton::clicked, this, &AdminDashboard::deleteContent);

        // Analytics Section
        analyticsTable = new QTableWidget(this);
        layout->addWidget(new QLabel("Analytics"));
        layout->addWidget(analyticsTable);

        QPushButton *refreshAnalyticsBtn = new QPushButton("Refresh Analytics", this);
        layout->addWidget(refreshAnalyticsBtn);
        connect(refreshAnalyticsBtn, &QPushButton::clicked, this, &AdminDashboard::refreshAnalytics);

        setLayout(layout);
    }

    // Function to add a user
    void addUser() {
        QString username = "User" + QString::number(rand() % 1000);
        dbManager->addUser(username, username + "@example.com", "admin");
        loadUserData();
    }

    // Function to delete content
    void deleteContent() {
        dbManager->getContentData();
        // For simplicity, assume we delete the first content
        QSqlQuery query("DELETE FROM content WHERE id=1");
        query.exec();
        loadContentData();
    }

    // Function to refresh analytics
    void refreshAnalytics() {
        loadAnalyticsData();
    }

    // Function to load user data into the UI
    void loadUserData() {
        QSqlQuery query = dbManager->getUserData();
        userTable->setRowCount(0);
        while (query.next()) {
            int row = userTable->rowCount();
            userTable->insertRow(row);
            userTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            userTable->setItem(row, 1, new QTableWidgetItem(query.value("username").toString()));
            userTable->setItem(row, 2, new QTableWidgetItem(query.value("email").toString()));
            userTable->setItem(row, 3, new QTableWidgetItem(query.value("role").toString()));
        }
    }

    // Function to load content data into the UI
    void loadContentData() {
        QSqlQuery query = dbManager->getContentData();
        contentTable->setRowCount(0);
        while (query.next()) {
            int row = contentTable->rowCount();
            contentTable->insertRow(row);
            contentTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            contentTable->setItem(row, 1, new QTableWidgetItem(query.value("user_id").toString()));
            contentTable->setItem(row, 2, new QTableWidgetItem(query.value("content").toString()));
            contentTable->setItem(row, 3, new QTableWidgetItem(query.value("date").toString()));
        }
    }

    // Function to load analytics data into the UI
    void loadAnalyticsData() {
        QSqlQuery query = dbManager->getAnalyticsData();
        analyticsTable->setRowCount(0);
        while (query.next()) {
            int row = analyticsTable->rowCount();
            analyticsTable->insertRow(row);
            analyticsTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            analyticsTable->setItem(row, 1, new QTableWidgetItem(query.value("user_id").toString()));
            analyticsTable->setItem(row, 2, new QTableWidgetItem(query.value("actions").toString()));
            analyticsTable->setItem(row, 3, new QTableWidgetItem(query.value("timestamp").toString()));
        }
    }
};
class AdminDashboard : public QWidget {
    Q_OBJECT
public:
    AdminDashboard() {
        setupUI();
        dbManager = new DatabaseManager();
    }

    ~AdminDashboard() {
        delete dbManager;
    }

private:
    DatabaseManager *dbManager;
    QTableWidget *userTable;
    QTableWidget *contentTable;
    QTableWidget *analyticsTable;

    void setupUI() {
        QVBoxLayout *layout = new QVBoxLayout(this);

        // User Management Section
        userTable = new QTableWidget(this);
        layout->addWidget(new QLabel("User Management"));
        layout->addWidget(userTable);

        QPushButton *addUserBtn = new QPushButton("Add User", this);
        layout->addWidget(addUserBtn);

        connect(addUserBtn, &QPushButton::clicked, this, &AdminDashboard::addUser);

        // Content Moderation Section
        contentTable = new QTableWidget(this);
        layout->addWidget(new QLabel("Content Moderation"));
        layout->addWidget(contentTable);

        QPushButton *deleteContentBtn = new QPushButton("Delete Content", this);
        layout->addWidget(deleteContentBtn);
        connect(deleteContentBtn, &QPushButton::clicked, this, &AdminDashboard::deleteContent);

        // Analytics Section
        analyticsTable = new QTableWidget(this);
        layout->addWidget(new QLabel("Analytics"));
        layout->addWidget(analyticsTable);

        QPushButton *refreshAnalyticsBtn = new QPushButton("Refresh Analytics", this);
        layout->addWidget(refreshAnalyticsBtn);
        connect(refreshAnalyticsBtn, &QPushButton::clicked, this, &AdminDashboard::refreshAnalytics);

        setLayout(layout);
    }

    // Function to add a user
    void addUser() {
        QString username = "User" + QString::number(rand() % 1000);
        dbManager->addUser(username, username + "@example.com", "admin");
        loadUserData();
    }

    // Function to delete content
    void deleteContent() {
        dbManager->getContentData();
        // For simplicity, assume we delete the first content
        QSqlQuery query("DELETE FROM content WHERE id=1");
        query.exec();
        loadContentData();
    }

    // Function to refresh analytics
    void refreshAnalytics() {
        loadAnalyticsData();
    }

    // Function to load user data into the UI
    void loadUserData() {
        QSqlQuery query = dbManager->getUserData();
        userTable->setRowCount(0);
        while (query.next()) {
            int row = userTable->rowCount();
            userTable->insertRow(row);
            userTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            userTable->setItem(row, 1, new QTableWidgetItem(query.value("username").toString()));
            userTable->setItem(row, 2, new QTableWidgetItem(query.value("email").toString()));
            userTable->setItem(row, 3, new QTableWidgetItem(query.value("role").toString()));
        }
    }

    // Function to load content data into the UI
    void loadContentData() {
        QSqlQuery query = dbManager->getContentData();
        contentTable->setRowCount(0);
        while (query.next()) {
            int row = contentTable->rowCount();
            contentTable->insertRow(row);
            contentTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            contentTable->setItem(row, 1, new QTableWidgetItem(query.value("user_id").toString()));
            contentTable->setItem(row, 2, new QTableWidgetItem(query.value("content").toString()));
            contentTable->setItem(row, 3, new QTableWidgetItem(query.value("date").toString()));
        }
    }

    // Function to load analytics data into the UI
    void loadAnalyticsData() {
        QSqlQuery query = dbManager->getAnalyticsData();
        analyticsTable->setRowCount(0);
        while (query.next()) {
            int row = analyticsTable->rowCount();
            analyticsTable->insertRow(row);
            analyticsTable->setItem(row, 0, new QTableWidgetItem(query.value("id").toString()));
            analyticsTable->setItem(row, 1, new QTableWidgetItem(query.value("user_id").toString()));
            analyticsTable->setItem(row, 2, new QTableWidgetItem(query.value("actions").toString()));
            analyticsTable->setItem(row, 3, new QTableWidgetItem(query.value("timestamp").toString()));
        }
    }
};


