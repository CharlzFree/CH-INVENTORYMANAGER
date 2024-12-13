# CH-INVENTORYMANAGER
# NOTE: Change the directory file path into your choosen folder at the file explorer (e.g., C:\Users\Admin\Your_Folder).
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

namespace RestaurantStorage
{
    public abstract class Product
    {
        public string Name { get; set; }
        public int Quantity { get; set; }
        public decimal Price { get; set; }
        public string QuantityUnit { get; set; }

        public Product(string name, int quantity, decimal price, string quantityUnit)
        {
            Name = name;
            Quantity = quantity;
            Price = price;
            QuantityUnit = quantityUnit;
        }

        public abstract void DisplayInfo();

        public void UpdateQuantity(int newQuantity)
        {
            Quantity = newQuantity;
            Console.WriteLine($"{Name} quantity updated to {Quantity} {QuantityUnit}.");
        }

        public void UpdatePrice(decimal newPrice)
        {
            Price = newPrice;
            Console.WriteLine($"{Name} price updated to {Price:C}.");
        }

        public void UpdateQuantityUnit(string newUnit)
        {
            QuantityUnit = newUnit;
            Console.WriteLine($"{Name} quantity unit updated to {QuantityUnit}.");
        }
    }

    public class Perishable : Product
    {
        public Perishable(string name, int quantity, decimal price, string quantityUnit)
            : base(name, quantity, price, quantityUnit) { }

        public override void DisplayInfo()
        {
            Console.WriteLine($"Perishable Product: {Name} | Quantity: {Quantity} {QuantityUnit} | Price: {Price:C}");
        }
    }

    public class NonPerishable : Product
    {
        public NonPerishable(string name, int quantity, decimal price, string quantityUnit)
            : base(name, quantity, price, quantityUnit) { }

        public override void DisplayInfo()
        {
            Console.WriteLine($"Non-Perishable Product: {Name} | Quantity: {Quantity} {QuantityUnit} | Price: {Price:C}");
        }
    }

    public class SalesReport
    {
        private readonly string salesFilePath;

        public SalesReport(string filePath)
        {
            salesFilePath = filePath;
        }

        public void RecordSale(Product product, int quantitySold)
        {
            decimal profit = product.Price * quantitySold;
            SaveSale(product, quantitySold, profit);
        }

        public void GenerateReport()
        {
            if (File.Exists(salesFilePath))
            {
                Console.WriteLine("\nSales Report:");
                Console.WriteLine($"{"Product",-20} | {"Quantity Sold",-15} | {"Profit",-10}");
                Console.WriteLine(new string('-', 50));

                foreach (var line in File.ReadLines(salesFilePath))
                {
                    var parts = line.Split(',');

                    if (parts.Length == 3 && int.TryParse(parts[1].Trim(), out int quantitySold) &&
                        decimal.TryParse(parts[2].Trim(), out decimal profit))
                    {
                        string productName = parts[0];
                        Console.WriteLine($"{productName,-20} | {quantitySold,-15} | {profit:C}");
                    }
                }
            }
            else
            {
                Console.WriteLine("No sales report data found.");
            }
        }

        private void SaveSale(Product product, int quantitySold, decimal profit)
        {
            using (StreamWriter writer = new StreamWriter(salesFilePath, true))
            {
                writer.WriteLine($"{product.Name},{quantitySold},{profit}");
            }
        }
    }

    class Program
    {
        private static List<Product> productInventory = new List<Product>();
        private static SalesReport salesReport;
        private static string inventoryFilePath;

        static void Main(string[] args)
        {
            Console.WriteLine("Welcome to the Storage Management System!");

            string directoryPath = @"C:\Users\Charlz\ProjectFilePath";
            if (!Directory.Exists(directoryPath))
            {
                Directory.CreateDirectory(directoryPath);
            }

            string[] inventoryFiles = Directory.GetFiles(directoryPath, "*.txt")
                .Where(file => !file.EndsWith("SalesReport.txt"))
                .ToArray();

            if (inventoryFiles.Length == 0)
            {
                Console.WriteLine("No existing inventory files found.");
                inventoryFiles = new string[] { };
            }

            Console.WriteLine("\nAvailable Inventory Files:");
            for (int i = 0; i < inventoryFiles.Length; i++)
            {
                Console.WriteLine($"{i + 1}. {Path.GetFileName(inventoryFiles[i])}");
            }

            Console.WriteLine("\nSelect an existing inventory file by number, or choose option to create a new one:");
            Console.WriteLine("0. Create a New Inventory File");

            Console.Write("Choose an option: ");
            int selectedFile;
            while (!int.TryParse(Console.ReadLine(), out selectedFile) || selectedFile < 0 || selectedFile > inventoryFiles.Length)
            {
                Console.WriteLine("Invalid input. Please enter a valid number.");
            }

            if (selectedFile == 0)
            {
                string newFileName;
                do
                {
                    Console.Write("Enter a new inventory file name (without extension): ");
                    newFileName = Console.ReadLine().Trim();
                    if (!newFileName.EndsWith(".txt"))
                    {
                        newFileName += ".txt";
                    }
                } while (File.Exists(Path.Combine(directoryPath, newFileName)));

                inventoryFilePath = Path.Combine(directoryPath, newFileName);
                File.Create(inventoryFilePath).Close();
                Console.WriteLine($"New inventory file {newFileName} created.");
            }
            else
            {
                inventoryFilePath = inventoryFiles[selectedFile - 1];
            }

            string salesFileName = $"SalesReport_{Path.GetFileName(inventoryFilePath)}";
            string salesFilePath = Path.Combine(directoryPath, salesFileName);
            salesReport = new SalesReport(salesFilePath);

            if (File.Exists(inventoryFilePath))
            {
                LoadProducts();
            }

            bool keepRunning = true;
            while (keepRunning)
            {
                Console.WriteLine("\nMenu:");
                Console.WriteLine("1. Add a Product");
                Console.WriteLine("2. Update Product");
                Console.WriteLine("3. Display All Products");
                Console.WriteLine("4. Search for a Product");
                Console.WriteLine("5. Record a Sale");
                Console.WriteLine("6. Generate Weekly Sales Report");
                Console.WriteLine("7. Exit");
                Console.Write("Choose an option (1-7): ");

                string choice = Console.ReadLine();
                switch (choice)
                {
                    case "1":
                        AddProduct();
                        break;
                    case "2":
                        UpdateProduct();
                        break;
                    case "3":
                        DisplayAllProducts();
                        break;
                    case "4":
                        SearchProduct();
                        break;
                    case "5":
                        RecordSale();
                        break;
                    case "6":
                        salesReport.GenerateReport();
                        break;
                    case "7":
                        SaveProducts();
                        keepRunning = false;
                        break;
                    default:
                        Console.WriteLine("Invalid option. Please try again.");
                        break;
                }
            }

            Console.WriteLine("Thank you for using the system. Goodbye!");
        }

        static void AddProduct()
        {
            Console.Write("\nEnter product name: ");
            string name = Console.ReadLine();

            Console.Write("Enter quantity: ");
            int quantity = int.Parse(Console.ReadLine());

            Console.Write("Enter price: ");
            decimal price = decimal.Parse(Console.ReadLine());

            Console.Write("Enter quantity unit: ");
            string quantityUnit = Console.ReadLine();

            Console.Write("Is this product perishable? (yes/no): ");
            string perishable = Console.ReadLine().ToLower();

            if (perishable == "yes")
            {
                productInventory.Add(new Perishable(name, quantity, price, quantityUnit));
            }
            else
            {
                productInventory.Add(new NonPerishable(name, quantity, price, quantityUnit));
            }

            SaveProducts();
            Console.WriteLine($"{name} has been added to the inventory.");
        }

        static void UpdateProduct()
        {
            Console.WriteLine("\nAvailable Products:");
            for (int i = 0; i < productInventory.Count; i++)
            {
                Console.WriteLine($"{i + 1}. {productInventory[i].Name}");
            }

            Console.Write("Select a product to update by number: ");
            int selectedProduct = int.Parse(Console.ReadLine()) - 1;

            if (selectedProduct >= 0 && selectedProduct < productInventory.Count)
            {
                Console.WriteLine("\nUpdate Options:");
                Console.WriteLine("1. Update Quantity");
                Console.WriteLine("2. Update Price");
                Console.WriteLine("3. Update Quantity Unit");

                Console.Write("Choose an option: ");
                string updateChoice = Console.ReadLine();

                switch (updateChoice)
                {
                    case "1":
                        Console.Write("Enter new quantity: ");
                        int newQuantity = int.Parse(Console.ReadLine());
                        productInventory[selectedProduct].UpdateQuantity(newQuantity);
                        break;

                    case "2":
                        Console.Write("Enter new price: ");
                        decimal newPrice = decimal.Parse(Console.ReadLine());
                        productInventory[selectedProduct].UpdatePrice(newPrice);
                        break;

                    case "3":
                        Console.Write("Enter new quantity unit: ");
                        string newUnit = Console.ReadLine();
                        productInventory[selectedProduct].UpdateQuantityUnit(newUnit);
                        break;

                    default:
                        Console.WriteLine("Invalid option.");
                        break;
                }
            }
            else
            {
                Console.WriteLine("Invalid selection.");
            }

            SaveProducts();
        }

        static void DisplayAllProducts()
        {
            Console.WriteLine("\nAll Products:");
            foreach (var product in productInventory)
            {
                product.DisplayInfo();
            }
        }

        static void SearchProduct()
        {
            Console.Write("\nEnter product name to search: ");
            string searchName = Console.ReadLine().ToLower();

            var foundProducts = productInventory.Where(p => p.Name.ToLower().Contains(searchName)).ToList();

            if (foundProducts.Any())
            {
                foreach (var product in foundProducts)
                {
                    product.DisplayInfo();
                }
            }
            else
            {
                Console.WriteLine("No matching products found.");
            }
        }

        static void RecordSale()
        {
            Console.Write("\nEnter product name for the sale: ");
            string productName = Console.ReadLine();

            var product = productInventory.FirstOrDefault(p => p.Name.Equals(productName, StringComparison.OrdinalIgnoreCase));

            if (product != null)
            {
                Console.Write("Enter quantity sold: ");
                int quantitySold = int.Parse(Console.ReadLine());

                if (quantitySold <= product.Quantity)
                {
                    salesReport.RecordSale(product, quantitySold);
                    product.UpdateQuantity(product.Quantity - quantitySold);
                }
                else
                {
                    Console.WriteLine("Not enough stock for this sale.");
                }
            }
            else
            {
                Console.WriteLine("Product not found.");
            }
        }

        static void LoadProducts()
        {
            if (File.Exists(inventoryFilePath))
            {
                var lines = File.ReadLines(inventoryFilePath);
                foreach (var line in lines)
                {
                    var parts = line.Split(',');

                    string name = parts[0];
                    int quantity = int.Parse(parts[1]);
                    decimal price = decimal.Parse(parts[2]);
                    string quantityUnit = parts[3];
                    string type = parts[4];

                    if (type == "Perishable")
                    {
                        productInventory.Add(new Perishable(name, quantity, price, quantityUnit));
                    }
                    else
                    {
                        productInventory.Add(new NonPerishable(name, quantity, price, quantityUnit));
                    }
                }
            }
        }

        static void SaveProducts()
        {
            using (StreamWriter writer = new StreamWriter(inventoryFilePath, false))
            {
                foreach (var product in productInventory)
                {
                    writer.WriteLine($"{product.Name},{product.Quantity},{product.Price},{product.QuantityUnit},{(product is Perishable ? "Perishable" : "NonPerishable")}");
                }
            }
        }
    }
}
