#include <iostream>
#include <vector>
#include <iomanip>
#include <string>
using namespace std;

/* ---------------- Q1 ---------------- */
class B {
public:
    virtual void VF() {
        cout << "B" << endl;
    }
};
class C1 : public B {
public:
    void VF() override {
        cout << "C" << endl;
    }
};
class D1 : public B {
public:
    void VF() override {
        cout << "D" << endl;
    }
};
void question1() {
    C1 objC;
    D1 objD;
    B* ptrB;

    ptrB = &objC;
    ptrB->VF();
    ptrB = &objD;
    ptrB->VF();

    C1* ptrC = &objC;
    D1* ptrD = &objD;
    ptrC->VF();
    ptrD->VF();
}

/* ---------------- Q2 ---------------- */
class Shape {
public:
    virtual double Area() = 0;
    virtual string getName() = 0;
    virtual ~Shape() {}
};
class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    double Area() override { return 3.14159 * radius * radius; }
    string getName() override { return "Circle"; }
};
class RectangleS : public Shape {
    double length, breadth;
public:
    RectangleS(double l, double b) : length(l), breadth(b) {}
    double Area() override { return length * breadth; }
    string getName() override { return "Rectangle"; }
};
void question2() {
    vector<Shape*> shapes;
    int choice;
    do {
        cout << "\nQ2 Menu:\n1. Add Circle\n2. Add Rectangle\n3. Display All\n4. Exit Q2\nEnter choice: ";
        cin >> choice;
        if (choice == 1) {
            double r;
            cout << "Enter radius: ";
            cin >> r;
            shapes.push_back(new Circle(r));
        } else if (choice == 2) {
            double l, b;
            cout << "Enter length and breadth: ";
            cin >> l >> b;
            shapes.push_back(new RectangleS(l, b));
        } else if (choice == 3) {
            cout << left << setw(15) << "Shape" << "Area\n";
            for (auto s : shapes) {
                cout << left << setw(15) << s->getName() << s->Area() << endl;
            }
        }
    } while (choice != 4);

    for (auto s : shapes) delete s;
}

/* ---------------- Q3 & Q4 ---------------- */
class ShapeBase {
protected:
    double x, y;
public:
    void get_data() {
        cin >> x >> y;
    }
    virtual void display_area() = 0;
    virtual ~ShapeBase() {}
};
class Triangle : public ShapeBase {
public:
    void display_area() override {
        cout << "Area of Triangle: " << 0.5 * x * y << endl;
    }
};
class RectangleB : public ShapeBase {
public:
    void display_area() override {
        cout << "Area of Rectangle: " << x * y << endl;
    }
};
void question3and4() {
    int choice;
    do {
        cout << "\nQ3 & Q4 Menu:\n1. Triangle\n2. Rectangle\n3. Exit Q3/Q4\nEnter choice: ";
        cin >> choice;
        ShapeBase* shape = nullptr;

        if (choice == 1) {
            cout << "Enter base and height: ";
            shape = new Triangle();
            shape->get_data();
            shape->display_area();
        } else if (choice == 2) {
            cout << "Enter length and breadth: ";
            shape = new RectangleB();
            shape->get_data();
            shape->display_area();
        }
        delete shape;
    } while (choice != 3);
}

/* ---------------- Q5 ---------------- */
class Sale {
protected:
    int itemQuantity;
    double unitPrice;
public:
    void setData(int qty, double price) {
        itemQuantity = qty;
        unitPrice = price;
    }
    double getBill() const {
        return itemQuantity * unitPrice;
    }
};
class DiwaliDiscountSale : public Sale {
    double discountPercent;
public:
    void setDiscount(double discount) {
        discountPercent = discount;
    }
    double computeSaving() const {
        return (getBill() * discountPercent) / 100.0;
    }
    double getFinalBill() const {
        return getBill() - computeSaving();
    }
};
void question5() {
    int qty;
    double price, discount;
    cout << "Enter number of items: ";
    cin >> qty;
    cout << "Enter unit price: ";
    cin >> price;
    cout << "Enter discount percent: ";
    cin >> discount;

    DiwaliDiscountSale sale;
    sale.setData(qty, price);
    sale.setDiscount(discount);

    cout << "Total Bill: " << sale.getBill() << endl;
    cout << "You Save: " << sale.computeSaving() << endl;
    cout << "Final Bill after Discount: " << sale.getFinalBill() << endl;
}

/* ---------------- Main Menu ---------------- */
int main() {
    int choice;
    do {
        cout << "\nMAIN MENU\n";
        cout << "1. Q1 - Virtual Function with Inheritance\n";
        cout << "2. Q2 - Shape Interface with Array of Shapes\n";
        cout << "3. Q3 & Q4 - Base Class Shape with Triangle and Rectangle\n";
        cout << "4. Q5 - Garment Sales with Diwali Discount\n";
        cout << "5. Exit\nEnter choice: ";
        cin >> choice;

        switch (choice) {
            case 1: question1(); break;
            case 2: question2(); break;
            case 3: question3and4(); break;
            case 4: question5(); break;
            case 5: cout << "Exiting program...\n"; break;
            default: cout << "Invalid choice.\n";
        }
    } while (choice != 5);

    return 0;
}
