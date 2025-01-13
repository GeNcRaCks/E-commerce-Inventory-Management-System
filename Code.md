# E-commerce-Inventory-Management-System

#include <iostream>
#include <string>
#include <queue>
#include <vector>
#include <algorithm>

using namespace std;

struct Product
{
    int id;
    string name;
    string category;
    float price;
    int stock;

    Product(int pid, string pname, string pcategory, float pprice, int pstock) : id(pid), name(pname), category(pcategory), price(pprice), stock(pstock) {}
};
struct TreeNode
{
    Product product;
    TreeNode* left;
    TreeNode* right;

    TreeNode(Product p) : product(p), left(nullptr), right(nullptr) {}
};
class Inventory
{
private:
    TreeNode* root;
    priority_queue<int, vector<int>, greater<int>> lowStockHeap;

    TreeNode* addProduct(TreeNode* node, Product product)
    {
        if (!node)
            return new TreeNode(product);
        if (product.name < node->product.name)
            node->left = addProduct(node->left, product);
        else
            node->right = addProduct(node->right, product);
        return node;
    }

    void inOrder(TreeNode* node, vector<Product>& products)
    {
        if (!node)
            return;
        inOrder(node->left, products);
        products.push_back(node->product);
        inOrder(node->right, products);
    }

    Product* searchById(TreeNode* node, int id)
    {
        if (!node)
            return nullptr;
        if (node->product.id == id)
            return &node->product;
        if (id < node->product.id)
            return searchById(node->left, id);
        else
            return searchById(node->right, id);
    }

public:
    Inventory() : root(nullptr) {}

    void addProduct(int id, string name, string category, float price, int stock)
    {
        Product newProduct(id, name, category, price, stock);
        root = addProduct(root, newProduct);
        if (stock < 5)
            lowStockHeap.push(id);
        cout << "Product added: " << name << endl;
    }

    void listProducts()
    {
        vector<Product> products;
        inOrder(root, products);
        if (products.empty())
        {
            cout << "No products in the inventory." << endl;
            return;
        }
        for (auto& product : products)
        {
            cout << "ID: " << product.id << ", Name: " << product.name
                << ", Category: " << product.category << ", Price: $"
                << product.price << ", Stock: " << product.stock << endl;
        }
    }

    Product* searchProduct(int id)
    {
        return searchById(root, id);
    }

    void checkLowStock()
    {
        cout << "Low stock products (Stock < 5):" << endl;
        if (lowStockHeap.empty())
        {
            cout << "No products with low stock." << endl;
            return;
        }
        while (!lowStockHeap.empty())
        {
            int id = lowStockHeap.top();
            lowStockHeap.pop();
            Product* product = searchProduct(id);
            if (product)
            {
                cout << "ID: " << product->id << ", Name: " << product->name
                    << ", Stock: " << product->stock << endl;
            }
        }
    }

    bool decrementStock(int productId)
    {
        Product* product = searchProduct(productId);
        if (product && product->stock > 0)
        {
            product->stock--;
            cout << "Stock decremented for Product ID: " << productId << ", New Stock: " << product->stock << endl;
            return true;
        }
        else
        {
            cout << "Product ID: " << productId << " is either out of stock or does not exist." << endl;
            return false;
        }
    }
};

class OrderQueue
{
private:
    queue<int> orderQueue;
    Inventory& inventory;

public:
    OrderQueue(Inventory& inv) : inventory(inv) {}

    void placeOrder(int productId)
    {
        orderQueue.push(productId);
        cout << "Order placed for product ID: " << productId << endl;
    }

    void processOrder()
    {
        if (orderQueue.empty())
        {
            cout << "No orders to process!" << endl;
            return;
        }
        int productId = orderQueue.front();
        orderQueue.pop();
        cout << "Processing order for product ID: " << productId << endl;
        inventory.decrementStock(productId);
    }
};

int main()
{
    Inventory inventory;
    OrderQueue orders(inventory);

    int choice;
    do
    {
        cout << "            E-Commerce Inventory Management" << endl;
        cout << "1. Add Product" << endl;
        cout << "2. List All Products" << endl;
        cout << "3. Search Product by ID" << endl;
        cout << "4. Check Low Stock Products" << endl;
        cout << "5. Place an Order" << endl;
        cout << "6. Process an Order" << endl;
        cout << "7. Exit" << endl;
        cout << endl;
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice)
        {
        case 1:
        {
            int id, stock;
            string name, category;
            float price;
            cout << "Enter Product ID: ";
            cin >> id;
            cin.ignore();
            cout << "Enter Product Name: ";
            getline(cin, name);
            cout << "Enter Product Category: ";
            getline(cin, category);
            cout << "Enter Product Price: ";
            cin >> price;
            cout << "Enter Product Stock: ";
            cin >> stock;
            inventory.addProduct(id, name, category, price, stock);
            break;
        }
        case 2:
            inventory.listProducts();
            break;
        case 3:
        {
            int id;
            cout << "Enter Product ID to Search: ";
            cin >> id;
            Product* product = inventory.searchProduct(id);
            if (product)
            {
                cout << "Found Product - ID: " << product->id
                    << ", Name: " << product->name << ", Stock: " << product->stock
                    << endl;
            }
            else
            {
                cout << "Product not found!" << endl;
            }
            break;
        }
        case 4:
            inventory.checkLowStock();
            break;
        case 5:
        {
            int productId;
            cout << "Enter Product ID to Place Order: ";
            cin >> productId;
            orders.placeOrder(productId);
            break;
        }
        case 6:
            orders.processOrder();
            break;
        case 7:
            cout << "Exiting the system" << endl;
            break;
        default:
            cout << "Invalid choice! Please try again." << endl;
        }
    } while (choice != 7);

    return 0;
}
