# DS-Project
#include <iostream>
#include <string>
#include <unordered_map>
#include <vector>
#include <cstdlib> // For system command

using namespace std;

// Utility function to clear the screen
void clearScreen() {
#ifdef _WIN32
    system("cls"); // For Windows
#else
    system("clear"); // For Unix-based systems
#endif
}

class User {
public:
    string username;
    string phoneNumber;
    string password;
    bool loggedIn;
    string feedback;

    User(string user, string phone, string pass) {
        username = user;
        phoneNumber = phone;
        password = pass;
        loggedIn = false;
        feedback = "";
    }
};

class Driver {
public:
    string username;
    string phoneNumber;
    string password;
    string vehicleNumber;
    bool loggedIn;
    bool isAvailable;

    Driver(string user, string phone, string pass, string vehicle) {
        username = user;
        phoneNumber = phone;
        password = pass;
        vehicleNumber = vehicle;
        loggedIn = false;
        isAvailable = true;
    }

    void updateStatus(bool status) {
        isAvailable = status;
    }
};

class Ride {
public:
    User* user;
    Driver* driver;
    string pickUpLocation;
    string dropOffLocation;
    bool rideCompleted;

    Ride(User* u, Driver* d, string pickUp, string dropOff)
        : user(u), driver(d), pickUpLocation(pickUp), dropOffLocation(dropOff), rideCompleted(false) {}

    void completeRide() {
        rideCompleted = true;
        cout << "Ride completed! Thank you for using our service." << endl;
    }

    void viewRideStatus() {
        if (rideCompleted) {
            cout << "Ride Status: Completed" << endl;
        } else {
            cout << "Ride Status: In Progress" << endl;
        }
    }

    void leaveFeedback() {
        if (rideCompleted) {
            cout << "Please leave feedback for the driver (1-5 stars): ";
            int feedback;
            cin >> feedback;
            user->feedback = to_string(feedback);
            cout << "Feedback for the driver: " << feedback << " stars." << endl;
        } else {
            cout << "You need to complete the ride before leaving feedback." << endl;
        }
    }
};

class Admin {
public:
    void viewUsers(const vector<User*>& users) {
        cout << "Users List:" << endl;
        for (const auto& user : users) {
            cout << "Username: " << user->username << " | Phone: " << user->phoneNumber << endl;
        }
    }

    void viewDrivers(const vector<Driver*>& drivers) {
        cout << "Drivers List:" << endl;
        for (const auto& driver : drivers) {
            cout << "Username: " << driver->username << " | Phone: " << driver->phoneNumber 
                 << " | Vehicle Number: " << driver->vehicleNumber 
                 << (driver->isAvailable ? " - Available" : " - Not Available") << endl;
        }
    }
};

int main() {
    vector<User*> users;
    vector<Driver*> drivers;
    unordered_map<string, string> userCredentials;
    unordered_map<string, string> driverCredentials;
    vector<Ride*> rides;

    Admin admin;
    User* loggedInUser = nullptr;
    Driver* loggedInDriver = nullptr;

    int choice;
    while (true) {
        clearScreen(); // Clear the screen before showing the main menu
        cout << "\n---- Main Menu ----" << endl;
        cout << "1. User Registration" << endl;
        cout << "2. Driver Registration" << endl;
        cout << "3. Admin Panel" << endl;
        cout << "4. User Login" << endl;
        cout << "5. Driver Login" << endl;
        cout << "6. Exit" << endl;
        cout << "Enter your choice: ";
        cin >> choice;

        if (choice == 1) {  // User Registration
            clearScreen();
            string username, phoneNumber, password;
            cout << "Enter Username: ";
            cin >> username;
            cout << "Enter Phone Number: ";
            cin >> phoneNumber;
            cout << "Enter Password: ";
            cin >> password;

            User* newUser = new User(username, phoneNumber, password);
            users.push_back(newUser);
            userCredentials[username] = password;

            cout << "User registered successfully!" << endl;
            system("pause");

        } else if (choice == 2) {  // Driver Registration
            clearScreen();
            string username, phoneNumber, password, vehicleNumber;
            cout << "Enter Username: ";
            cin >> username;
            cout << "Enter Phone Number: ";
            cin >> phoneNumber;
            cout << "Enter Password: ";
            cin >> password;
            cout << "Enter Vehicle Number: ";
            cin >> vehicleNumber;

            Driver* newDriver = new Driver(username, phoneNumber, password, vehicleNumber);
            drivers.push_back(newDriver);
            driverCredentials[username] = password;

            cout << "Driver registered successfully!" << endl;
            system("pause");

        } else if (choice == 3) {  // Admin Panel
            while (true) {
                clearScreen();
                int adminChoice;
                cout << "\n--- Admin Panel ---" << endl;
                cout << "1. View Users" << endl;
                cout << "2. View Drivers" << endl;
                cout << "3. Back to Main Menu" << endl;
                cout << "Enter your choice: ";
                cin >> adminChoice;

                if (adminChoice == 1) {
                    clearScreen();
                    admin.viewUsers(users);
                    system("pause");
                } else if (adminChoice == 2) {
                    clearScreen();
                    admin.viewDrivers(drivers);
                    system("pause");
                } else if (adminChoice == 3) {
                    break;
                }
            }

        } else if (choice == 4) {  // User Login
            clearScreen();
            string username, password;
            cout << "Enter Username: ";
            cin >> username;
            cout << "Enter Password: ";
            cin >> password;

            if (userCredentials.find(username) != userCredentials.end() && userCredentials[username] == password) {
                loggedInUser = nullptr;
                for (auto user : users) {
                    if (user->username == username) {
                        loggedInUser = user;
                        break;
                    }
                }

                if (loggedInUser != nullptr) {
                    loggedInUser->loggedIn = true;
                    while (true) {
                        clearScreen();
                        cout << "\n---- User Panel ----" << endl;
                        cout << "1. Request a Ride" << endl;
                        cout << "2. View Ride Status" << endl;
                        cout << "3. Provide Feedback" << endl;
                        cout << "4. Logout" << endl;
                        cout << "Enter your choice: ";
                        int userChoice;
                        cin >> userChoice;

                        if (userChoice == 1) {
                            clearScreen();
                            string pickUp, dropOff;
                            cout << "Enter Pick-Up Location: ";
                            cin.ignore();
                            getline(cin, pickUp);
                            cout << "Enter Drop-Off Location: ";
                            getline(cin, dropOff);

                            bool rideRequested = false;
                            for (auto driver : drivers) {
                                if (driver->isAvailable) {
                                    rideRequested = true;
                                    driver->isAvailable = false;

                                    Ride* ride = new Ride(loggedInUser, driver, pickUp, dropOff);
                                    rides.push_back(ride);
                                    cout << "Ride requested successfully! Driver with username " 
                                         << driver->username << " will pick you up." << endl;
                                    break;
                                }
                            }

                            if (!rideRequested) {
                                cout << "No available drivers at the moment. Please try again later." << endl;
                            }
                            system("pause");

                        } else if (userChoice == 2) {
                            clearScreen();
                            if (!rides.empty()) {
                                rides.back()->viewRideStatus();
                            } else {
                                cout << "No rides found!" << endl;
                            }
                            system("pause");

                        } else if (userChoice == 3) {
                            clearScreen();
                            if (!rides.empty()) {
                                rides.back()->leaveFeedback();
                            } else {
                                cout << "No rides to leave feedback for!" << endl;
                            }
                            system("pause");

                        } else if (userChoice == 4) {
                            loggedInUser->loggedIn = false;
                            break;
                        }
                    }
                }
            } else {
                cout << "Invalid login credentials." << endl;
                system("pause");
            }

        } else if (choice == 5) {  // Driver Login
            clearScreen();
            string username, password;
            cout << "Enter Username: ";
            cin >> username;
            cout << "Enter Password: ";
            cin >> password;

            if (driverCredentials.find(username) != driverCredentials.end() && driverCredentials[username] == password) {
                loggedInDriver = nullptr;
                for (auto driver : drivers) {
                    if (driver->username == username) {
                        loggedInDriver = driver;
                        break;
                    }
                }

                if (loggedInDriver != nullptr) {
                    loggedInDriver->loggedIn = true;
                    while (true) {
                        clearScreen();
                        cout << "\n---- Driver Panel ----" << endl;
                        cout << "1. Update Availability" << endl;
                        cout << "2. View Rides" << endl;
                        cout << "3. Logout" << endl;
                        cout << "Enter your choice: ";
                        int driverChoice;
                        cin >> driverChoice;

                        if (driverChoice == 1) {
                            bool status;
                            cout << "Enter availability status (1 for available, 0 for not available): ";
                            cin >> status;
                            loggedInDriver->updateStatus(status);
                            cout << "Driver status updated." << endl;
                            system("pause");

                        } else if (driverChoice == 2) {
                            clearScreen();
                            cout << "\n---- Associated Rides ----" << endl;
                            for (auto ride : rides) {
                                if (ride->driver == loggedInDriver) {
                                    cout << "Ride for User: " << ride->user->username << endl;
                                    cout << "Pickup: " << ride->pickUpLocation << endl;
                                    cout << "Dropoff: " << ride->dropOffLocation << endl;
                                    ride->viewRideStatus();
                                    if (!ride->rideCompleted) {
                                        cout << "Complete this ride? (1 for Yes, 0 for No): ";
                                        int complete;
                                        cin >> complete;
                                        if (complete == 1) {
                                            ride->completeRide();
                                            ride->driver->isAvailable = true;
                                        }
                                    }
                                    cout << "-----------------------------------" << endl;
                                }
                            }
                            system("pause");

                        } else if (driverChoice == 3) {
                            loggedInDriver->loggedIn = false;
                            break;
                        }
                    }
                }
            } else {
                cout << "Invalid login credentials." << endl;
                system("pause");
            }

        } else if (choice == 6) {
            cout << "Exiting program." << endl;
            system("pause");
            break;
        } else {
            cout << "Invalid choice. Please try again." << endl;
            system("pause");
        }
    }

    // Clean up dynamically allocated memory
    for (auto user : users) delete user;
    for (auto driver : drivers) delete driver;
    for (auto ride : rides) delete ride;

    return 0;
}
