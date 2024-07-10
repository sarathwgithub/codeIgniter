
1. Set Up Your MySQL Database
    ```sql
        CREATE TABLE products (
          id INT AUTO_INCREMENT PRIMARY KEY,
          name VARCHAR(100) NOT NULL,
          description TEXT,
          created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
          updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
      );
      
      CREATE TABLE stock (
          id INT AUTO_INCREMENT PRIMARY KEY,
          product_id INT NOT NULL,
          quantity INT NOT NULL,
          price DECIMAL(10, 2) NOT NULL,
          date_received DATE NOT NULL,
          created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
          updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
          FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
      );
      
      INSERT INTO products (name, description) VALUES
      ('Product A', 'Description for Product A'),
      ('Product B', 'Description for Product B'),
      ('Product C', 'Description for Product C');
      
      INSERT INTO stock (product_id, quantity, price, date_received) VALUES
      (1, 10, 100.00, '2024-07-01'),
      (2, 5, 50.00, '2024-07-01'),
      (3, 20, 75.00, '2024-07-01');

2. Create ProductsModel.php (app/Models/ProductsModel.php):    
    php spark make:model ProductsModel
      ```php
          namespace App\Models;
  
          use CodeIgniter\Model;
          
          class ProductsModel extends Model
          {
              protected $table = 'products';
              protected $primaryKey = 'id';
              protected $allowedFields = ['name', 'description', 'created_at', 'updated_at'];
              protected $useTimestamps = true;
          }
    
 
4. Create StockModel.php (app/Models/StockModel.php):
   php spark make:model StockModel
    ```php
      namespace App\Models;

      use CodeIgniter\Model;
      
      class StockModel extends Model
      {
          protected $table = 'stock';
          protected $primaryKey = 'id';
          protected $allowedFields = ['product_id', 'quantity', 'price', 'date_received', 'created_at', 'updated_at'];
          protected $useTimestamps = true;
      }
6. Create Product.php (app/Controllers/Product.php):
   php spark make:controller Product
    ```php
        <?php

        namespace App\Controllers;
        
        use App\Controllers\BaseController;
        use CodeIgniter\HTTP\ResponseInterface;
        use App\Models\ProductsModel;
        use App\Models\StockModel;
        use CodeIgniter\Controller;
        
        class Products extends BaseController
        {
            public function index()
            {
                $productModel = new ProductsModel();
                $data['products'] = $productModel->findAll();
        
                return view('products/index', $data);
            }
        
            public function create()
            {
                helper('form');
                return view('products/create');
            }
        
            public function store()
            {
                $productModel = new ProductsModel();
        
                $data = [
                    'name'        => $this->request->getPost('name'),
                    'description' => $this->request->getPost('description'),
                ];
        
                $productModel->save($data);
        
                return redirect()->to('/products');
            }
        
            public function edit($id)
            {
                helper('form');
                $productModel = new ProductsModel();
                $data['product'] = $productModel->find($id);
        
                return view('products/edit', $data);
            }
        
            public function update()
            {
                $productModel = new ProductsModel();
        
                $data = [
                    'name'        => $this->request->getPost('name'),
                    'description' => $this->request->getPost('description'),
                ];
        
                $productModel->update($this->request->getPost('id'), $data);
        
                return redirect()->to('/products');
            }
        
            public function delete($id)
            {
                $productModel = new ProductsModel();
                $productModel->delete($id);
        
                return redirect()->to('/products');
            }
        }

8. Create Stock.php (app/Controllers/Stock.php):
   php spark make:controller Stock
    ```php
        namespace App\Controllers;

        use App\Models\StockModel;
        use App\Models\ProductsModel;
        use CodeIgniter\Controller;
        
        class Stock extends BaseController
        {
            public function index()
            {
                $stockModel = new StockModel();
                $productModel = new ProductsModel();
                
                $data['stocks'] = $stockModel->select('stock.*, products.name')
                                             ->join('products', 'products.id = stock.product_id')
                                             ->findAll();
        
                return view('stock/index', $data);
            }
        
            public function create()
            {
                helper('form');
                $productModel = new ProductsModel();
                $data['products'] = $productModel->findAll();
        
                return view('stock/create', $data);
            }
        
            public function store()
            {
                $stockModel = new StockModel();
        
                $data = [
                    'product_id'   => $this->request->getPost('product_id'),
                    'quantity'     => $this->request->getPost('quantity'),
                    'price'        => $this->request->getPost('price'),
                    'date_received'=> $this->request->getPost('date_received'),
                ];
        
                $stockModel->save($data);
        
                return redirect()->to('/stock');
            }
        
            public function edit($id)
            {
                helper('form');
                $stockModel = new StockModel();
                $productModel = new ProductsModel();
        
                $data['stock'] = $stockModel->find($id);
                $data['products'] = $productModel->findAll();
        
                return view('stock/edit', $data);
            }
        
            public function update()
            {
                $stockModel = new StockModel();
        
                $data = [
                    'product_id'   => $this->request->getPost('product_id'),
                    'quantity'     => $this->request->getPost('quantity'),
                    'price'        => $this->request->getPost('price'),
                    'date_received'=> $this->request->getPost('date_received'),
                ];
        
                $stockModel->update($this->request->getPost('id'), $data);
        
                return redirect()->to('/stock');
            }
        
            public function delete($id)
            {
                $stockModel = new StockModel();
                $stockModel->delete($id);
        
                return redirect()->to('/stock');
            }
        }
10. Create index.php (app/Views/products/index.php)
    ```php
        <!DOCTYPE html>
        <html>
        <head>
            <title>Products</title>
        </head>
        <body>
            <h1>Products</h1>
            <a href="<?= site_url('products/create') ?>">Add New Product</a>
            <table border="1">
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Description</th>
                    <th>Actions</th>
                </tr>
                <?php foreach ($products as $product): ?>
                <tr>
                    <td><?= $product['id'] ?></td>
                    <td><?= $product['name'] ?></td>
                    <td><?= $product['description'] ?></td>
                    <td>
                        <a href="<?= site_url('products/edit/'.$product['id'])  ?>">Edit</a>
                        <a href="<?= site_url('products/delete/'.$product['id']) ?>" onclick="return confirm('Are you sure you want to delete this stock entry?')">Delete</a>
                    </td>
                </tr>
                <?php endforeach; ?>
            </table>
        </body>
        </html>



11. Create create.php (app/Views/products/create.php)
    ```php
        <!DOCTYPE html>
        <html>
        <head>
            <title>Add New Product</title>
        </head>
        <body>
            <h1>Add New Product</h1>
            <?= form_open('products/store') ?>
            
                <label>Name</label>
                <input type="text" name="name" required>
                <label>Description</label>
                <textarea name="description" required></textarea>
                <button type="submit">Save</button>
            <?= form_close() ?>
        </body>
        </html>


12. Create edit.php (app/Views/products/edit.php):
    ```html
        <!DOCTYPE html>
        <html>
        
        <head>
            <title>Edit Product</title>
        </head>
        
        <body>
            <h1>Edit Product</h1>
            <?= form_open('products/update') ?>
            <label>Name</label>
            <input type="text" name="name" value="<?= $product['name'] ?>" required>
            <label>Description</label>
            <textarea name="description" required><?= $product['description'] ?></textarea>
            <input type="hidden" name="id" id="id" value="<?= $product['id'] ?>">
            <button type="submit">Update</button>
            <?= form_close() ?>
        </body>
        
        </html>

13. Create index.php (app/Views/stock/index.php)
    ```html
        <!DOCTYPE html>
        <html>
        <head>
            <title>Stock</title>
        </head>
        <body>
            <h1>Stock</h1>
            <a href="<?= site_url('stock/create') ?>">Add New Stock</a>
            <table border="1">
                <tr>
                    <th>ID</th>
                    <th>Product Name</th>
                    <th>Quantity</th>
                    <th>Price</th>
                    <th>Date Received</th>
                    <th>Actions</th>
                </tr>
                <?php foreach ($stocks as $stock): ?>
                <tr>
                    <td><?= $stock['id'] ?></td>
                    <td><?= $stock['name'] ?></td>
                    <td><?= $stock['quantity'] ?></td>
                    <td><?= $stock['price'] ?></td>
                    <td><?= $stock['date_received'] ?></td>
                    <td>
                        <a href="<?= site_url('stock/edit/' . $stock['id']) ?>">Edit</a>
                        <a href="<?= site_url('stock/delete/' . $stock['id']) ?>" onclick="return confirm('Are you sure you want to delete this stock entry?')">Delete</a>
                    </td>
                </tr>
                <?php endforeach; ?>
            </table>
        </body>
        </html>


14. Create create.php (app/Views/stock/create.php)
    ```html
        <!DOCTYPE html>
        <html>
        <head>
            <title>Add New Stock</title>
        </head>
        <body>
            <h1>Add New Stock</h1>
            <?= form_open('stock/store') ?>    
                <label>Product</label>
                <select name="product_id" required>
                    <?php foreach ($products as $product): ?>
                        <option value="<?= $product['id'] ?>"><?= $product['name'] ?></option>
                    <?php endforeach; ?>
                </select>
                <label>Quantity</label>
                <input type="number" name="quantity" required>
                <label>Price</label>
                <input type="text" name="price" required>
                <label>Date Received</label>
                <input type="date" name="date_received" required>
                <button type="submit">Save</button>
                <?= form_close() ?>
        </body>
        </html>


15. Create edit.php (app/Views/stock/edit.php)
    
     ```html
            <!DOCTYPE html>
            <html>
            <head>
                <title>Edit Stock</title>
            </head>
            <body>
                <h1>Edit Stock</h1>
                
                <?= form_open('stock/update') ?>    
                    <label>Product</label>
                    <select name="product_id" required>
                        <?php foreach ($products as $product): ?>
                            <option value="<?= $product['id'] ?>" <?= ($stock['product_id'] == $product['id']) ? 'selected' : '' ?>><?= $product['name'] ?></option>
                        <?php endforeach; ?>
                    </select>
                    <label>Quantity</label>
                    <input type="number" name="quantity" value="<?= $stock['quantity'] ?>" required>
                    <label>Price</label>
                    <input type="text" name="price" value="<?= $stock['price'] ?>" required>
                    <label>Date Received</label>
                    <input type="date" name="date_received" value="<?= $stock['date_received'] ?>" required>
                    <input type="hidden" id="id" name="id" value="<?= $stock['id'] ?>">
                    <button type="submit">Update</button>
                    <?= form_close() ?>
            </body>
            </html>   

    
    
