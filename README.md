#include <iostream>
#include <vector>
#include <string>
#include <cmath>
#include <chrono>
#include <thread>
#include <cctype> // Required for tolower()

// --- Data Structures ---

/**
 * @brief Represents a simple 3D vector for position and direction.
 */
struct Vector3 {
    float x, y, z;

    // Default constructor
    Vector3(float x = 0.0f, float y = 0.0f, float z = 0.0f) : x(x), y(y), z(z) {}

    // Basic vector addition
    Vector3 operator+(const Vector3& other) const {
        return Vector3(x + other.x, y + other.y, z + other.z);
    }
    // Basic vector multiplication by a scalar
    Vector3 operator*(float scalar) const {
        return Vector3(x * scalar, y * scalar, z * scalar);
    }
};

// --- Character Class (The Player) ---

/**
 * @brief Represents the player character with position and movement logic.
 */
class Character {
private:
    Vector3 position;
    // Set movement speed to 1.0f for a clear 1-unit move per turn in this console simulation
    float movementSpeed = 1.0f; 
    float yaw = 0.0f;           // Rotation around the Y-axis (left/right look)

public:
    Character(Vector3 startPos) : position(startPos) {
        std::cout << "Character initialized at (" << position.x << ", " << position.y << ", " << position.z << ")\n";
    }

    /**
     * @brief Handles movement based on input (W/A/S/D equivalent).
     * @param direction: A normalized vector representing the intended movement direction.
     * @param deltaTime: Time elapsed since the last update (fixed at 1.0 for console turns).
     */
    void Move(const Vector3& direction, float deltaTime) {
        // Calculate the distance to move
        Vector3 displacement = direction * (movementSpeed * deltaTime);
        
        // Simple movement update 
        position = position + displacement;
    }

    /**
     * @brief Processes the console input command and updates character state.
     * @param deltaTime: Time elapsed since the last frame (fixed at 1.0 for console turns).
     * @param inputCommand: The key pressed by the user ('w', 'a', 's', 'd').
     */
    void Update(float deltaTime, char inputCommand) {
        Vector3 direction(0.0f, 0.0f, 0.0f);
        
        // Map console input to 3D movement direction (X/Z plane)
        switch (inputCommand) {
            case 'w': // Forward (Positive Z)
                direction.z = 1.0f;
                break;
            case 's': // Backward (Negative Z)
                direction.z = -1.0f;
                break;
            case 'a': // Left (Negative X)
                direction.x = -1.0f;
                break;
            case 'd': // Right (Positive X)
                direction.x = 1.0f;
                break;
            default:
                // Do nothing if input is not a movement key
                return; 
        }

        // Apply movement
        Move(direction, deltaTime);

        // Print position update, note the use of std::flush to ensure immediate output
        std::cout << "Character Pos: (" << position.x << ", " << position.y << ", " << position.z << ")\r" << std::flush;
    }

    /**
     * @brief Retrieves the character's current 3D position.
     * @return Vector3 The current position.
     */
    const Vector3& GetPosition() const {
        return position;
    }
};

// --- Game Engine Core ---

/**
 * @brief The main game class responsible for managing game state and the loop.
 */
class Game {
private:
    Character player;
    bool isRunning = true;
    char lastInput = ' '; // To store the last character command

    /**
     * @brief Initializes the graphics subsystem (now a console message).
     */
    void InitializeGraphics() {
        std::cout << "\n[Engine] Initializing console simulation...\n";
    }

    /**
     * @brief Handles user input by reading a character from the console.
     * This function blocks until a character is entered.
     */
    void ProcessInput() {
        std::cout << "\n[Turn] Enter command (w/a/s/d to move, q to quit): ";
        char input;
        
        // Wait for user input
        if (!(std::cin >> input)) {
            // Handle EOF or read error
            isRunning = false;
            lastInput = 'q';
            return;
        }
        lastInput = std::tolower(input); // Store and convert to lowercase
    }

    /**
     * @brief Updates all game objects (physics, AI, game logic).
     * @param deltaTime: Time since the last update (fixed at 1.0 for turn-based).
     */
    void Update(float deltaTime) {
        // Update the player character based on the collected input
        player.Update(deltaTime, lastInput);

        // Example condition to stop the loop
        if (player.GetPosition().z > 20.0f) {
            isRunning = false; 
            std::cout << "\n[Engine] Game loop exiting (target distance reached).\n";
        }
    }

    /**
     * @brief Draws the 3D scene (now a simple placeholder).
     */
    void DrawScene() {
        // In a real 3D engine, the scene would be rendered here.
    }

public:
    /**
     * @brief Game constructor.
     */
    Game() : player(Vector3(0.0f, 0.0f, 0.0f)) { 
        // Initial position reset to (0, 0, 0) for cleaner start
    }

    /**
     * @brief The main entry point for the game loop (now turn-based).
     */
    void Run() {
        InitializeGraphics();
        
        std::cout << "[Engine] Starting Interactive Console Simulation...\n";
        std::cout << "--- Move 1 unit per command in the X/Z plane ---\n";

        // Main loop structure
        while (isRunning) {
            // Delta time is fixed at 1.0 for simplicity in this turn-based console environment.
            float deltaTime = 1.0f; 

            // 1. Input (waits for you to enter a command)
            ProcessInput();
            
            if (lastInput == 'q') {
                isRunning = false;
                continue;
            }

            // 2. Update (Logic)
            Update(deltaTime);

            // 3. Render (Graphics - placeholder)
            DrawScene();
        }
        
        std::cout << "\n[Engine] Simulation finished. Thanks for playing!\n";
    }
};

/**
 * @brief Entry point of the C++ program.
 */
int main() {
    // Set up standard output formatting for better console display
    std::cout << std::fixed << std::setprecision(1); // Set precision to 1 decimal place

    try {
        Game myGame;
        myGame.Run();
    } catch (const std::exception& e) {
        std::cerr << "An error occurred: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
