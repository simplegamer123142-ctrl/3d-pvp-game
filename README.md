# 3d-pvp-game
#include <iostream>
#include <cmath>

/**
 * @brief A basic class representing a 3D vector (x, y, z).
 * * This class is fundamental for 3D graphics and physics, handling
 * positions, directions, and forces.
 */
class Vector3 {
public:
    double x, y, z;

    /**
     * @brief Default constructor. Initializes vector to (0, 0, 0).
     */
    Vector3() : x(0.0), y(0.0), z(0.0) {}

    /**
     * @brief Constructor with custom values.
     * @param _x The x-component.
     * @param _y The y-component.
     * @param _z The z-component.
     */
    Vector3(double _x, double _y, double _z) : x(_x), y(_y), z(_z) {}

    // --- Vector Operations ---

    /**
     * @brief Calculates the magnitude (length) of the vector.
     * @return The magnitude of the vector.
     */
    double magnitude() const {
        return std::sqrt(x * x + y * y + z * z);
    }

    /**
     * @brief Returns a new vector that is the normalized (unit) version of this vector.
     * * A unit vector has a magnitude of 1, preserving the original direction.
     * @return A normalized Vector3.
     */
    Vector3 normalize() const {
        double mag = magnitude();
        if (mag > 1e-6) { // Avoid division by zero
            return Vector3(x / mag, y / mag, z / mag);
        }
        return Vector3(); // Returns zero vector if magnitude is zero
    }

    /**
     * @brief Calculates the dot product with another vector.
     * @param other The other Vector3.
     * @return The scalar dot product value.
     */
    double dot(const Vector3& other) const {
        return x * other.x + y * other.y + z * other.z;
    }

    /**
     * @brief Calculates the cross product with another vector.
     * @param other The other Vector3.
     * @return The resulting Vector3 from the cross product.
     */
    Vector3 cross(const Vector3& other) const {
        return Vector3(
            y * other.z - z * other.y,
            z * other.x - x * other.z,
            x * other.y - y * other.x
        );
    }

    // --- Operator Overloads for Convenience ---

    /**
     * @brief Overloads the addition operator (+).
     * @param other The vector to add.
     * @return A new Vector3 that is the sum of the two vectors.
     */
    Vector3 operator+(const Vector3& other) const {
        return Vector3(x + other.x, y + other.y, z + other.z);
    }

    /**
     * @brief Overloads the subtraction operator (-).
     * @param other The vector to subtract.
     * @return A new Vector3 that is the difference of the two vectors.
     */
    Vector3 operator-(const Vector3& other) const {
        return Vector3(x - other.x, y - other.y, z - other.z);
    }

    /**
     * @brief Overloads the scalar multiplication operator (*).
     * @param scalar The scalar value to multiply by.
     * @return A new Vector3 scaled by the scalar.
     */
    Vector3 operator*(double scalar) const {
        return Vector3(x * scalar, y * scalar, z * scalar);
    }

    /**
     * @brief Overloads the stream insertion operator for easy printing.
     * @param os The output stream.
     * @param v The Vector3 to print.
     * @return The output stream reference.
     */
    friend std::ostream& operator<<(std::ostream& os, const Vector3& v) {
        os << "Vector3(" << v.x << ", " << v.y << ", " << v.z << ")";
        return os;
    }
};

/**
 * @brief Main function to demonstrate the Vector3 class.
 */
int main() {
    // 1. Initialization and printing
    Vector3 position(1.0, 2.0, 3.0);
    Vector3 direction(4.0, 0.0, 0.0);
    std::cout << "--- Vector3 Demonstration ---" << std::endl;
    std::cout << "Position: " << position << std::endl;
    std::cout << "Direction: " << direction << std::endl;

    // 2. Vector Addition and Subtraction
    Vector3 result_add = position + direction;
    std::cout << "Position + Direction: " << result_add << std::endl; // Expected: (5, 2, 3)

    // 3. Scalar Multiplication
    Vector3 scaled_pos = position * 2.5;
    std::cout << "Scaled Position: " << scaled_pos << std::endl; // Expected: (2.5, 5.0, 7.5)

    // 4. Magnitude and Normalization
    std::cout << "Magnitude of Position: " << position.magnitude() << std::endl; 
    Vector3 normalized_dir = direction.normalize();
    std::cout << "Normalized Direction: " << normalized_dir << std::endl; // Expected: (1, 0, 0)
    std::cout << "Magnitude of Normalized Direction: " << normalized_dir.magnitude() << std::endl;

    // 5. Dot Product (measures how similar directions are)
    Vector3 up(0.0, 1.0, 0.0);
    double dot_prod = up.dot(direction);
    std::cout << "Dot product (Up . Direction): " << dot_prod << std::endl; // Expected: 0 (perpendicular)

    // 6. Cross Product (measures perpendicular vector/rotation)
    Vector3 forward(1.0, 0.0, 0.0);
    Vector3 right = forward.cross(up);
    std::cout << "Cross product (Forward x Up): " << right << std::endl; // Expected: (0, 0, 1)

    return 0;
}
