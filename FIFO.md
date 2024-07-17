----------------------------------------------------------------
**To implement the FIFO issuing of items in CodeIgniter 4**
----------------------------------------------------------------

Step 01: Update Stock Model
  ```php
      protected $allowedFields    = ['product_id', 'quantity', 'price', 'date_received','issue_qty', 'created_at', 'updated_at'];
  ```

Step 02 : Add Issue Model
  ```php
      <?php
      // app/Models/IssueModel.php
      namespace App\Models;
      
      use CodeIgniter\Model;
      
      class IssueModel extends Model
      {
          protected $table = 'issue';
          protected $primaryKey = 'id';
          protected $allowedFields = ['order_id', 'item_id', 'stock_id', 'issue_qty', 'selling_price'];
      }
      
      ?>
  ```

Step 03: Update Stock Controller
  ```php
      use App\Models\IssueModel;

      public function getAvailableStock($item_id)
      {
          $stockModel = new StockModel();
          return $stockModel->where('product_id', $item_id)
              ->orderBy('date_received', 'ASC')
              ->findAll();
          //echo $stockModel->getLastQuery();
      }
  
      public function updateIssuedQuantity($stock_id, $issue_qty)
      {
          $stockModel = new StockModel();
          $stockModel->where('id', $stock_id)
              ->set('issue_qty', 'COALESCE(issue_qty, 0) + ' . $issue_qty, false)
              ->update();
      }
  
      public function issue()
      {
          helper('form');
  
          return view('stock/issue');
      }
      public function issueItem()
      {
          $itemModel = new ProductsModel();
          $itemStockModel = new StockModel();
          $issueModel = new IssueModel();
  
          $item_id = $this->request->getPost('item_id');
          $order_id = $this->request->getPost('order_id');
          $quantity = $this->request->getPost('quantity');
          $selling_price = $this->request->getPost('selling_price');
  
          $stocks = $this->getAvailableStock($item_id);
  
          $remainingQuantity = $quantity;
          $issued = [];
  
          foreach ($stocks as $stock) {
              if ($remainingQuantity <= 0) {
                  break;
              }
  
              $availableQty = $stock['quantity'] - $stock['issue_qty'];
              if ($availableQty <= 0) {
                  continue;
              }
  
              $issueQty = min($availableQty, $remainingQuantity);
  
              // Create issue record
              $issueData = [
                  'order_id' => $order_id,
                  'item_id' => $item_id,
                  'stock_id' => $stock['id'],
                  'issue_qty' => $issueQty,
                  'selling_price' => $selling_price
              ];
              $issueModel->insert($issueData);
  
              // Update stock issued quantity
              $this->updateIssuedQuantity($stock['id'], $issueQty);
  
              $remainingQuantity -= $issueQty;
              $issued[] = $issueData;
          }
              
          return view('stock/issue_result', [
              'issued' => $issued,
              'requested_quantity' => $quantity,
              'remaining_quantity' => $remainingQuantity
          ]);
      }
  ```
Step 04: Create issue table
  ```sql
      CREATE TABLE `issue` (
          `id` INT NOT NULL AUTO_INCREMENT,
          `order_id` INT NOT NULL,
          `item_id` INT NOT NULL,
          `stock_id` INT NOT NULL,
          `issue_qty` INT NOT NULL,
          `selling_price` DECIMAL(10, 2) NOT NULL,
          PRIMARY KEY (`id`)
         
      );
  ```
Step 05: Create Issue View
  ```html
      <!-- Views/stock/issue.php -->
      <!DOCTYPE html>
      <html>
      
      <head>
          <title>Issue Item</title>
      </head>
      
      <body>
          <h1>Issue Item</h1>
          <?= form_open('stock/issueItem') ?>
          <label for="item_id">Item ID:</label>
          <input type="number" name="item_id" id="item_id" required><br>
          <label for="order_id">Order ID:</label>
          <input type="number" name="order_id" id="order_id" required><br>
          <label for="quantity">Quantity:</label>
          <input type="number" name="quantity" id="quantity" required><br>
          <label for="selling_price">Selling Price:</label>
          <input type="number" name="selling_price" id="selling_price" required><br>
          <button type="submit">Issue</button>
          </form>
      </body>
      
      </html>
  ```
Step 06: Create Issue Result View
  ```html
      <!-- Views/stock/issue_result.php -->
      <!DOCTYPE html>
      <html lang="en">
      <head>
          <meta charset="UTF-8">
          <title>Stock Issue Result</title>
      </head>
      <body>
          <h1>Stock Issue Result</h1>
      
          <?php if ($remaining_quantity > 0): ?>
              <h2>Stock Issue Error</h2>
              <p>Not enough stock available to fulfill the requested quantity of <?= $requested_quantity ?>.</p>
              <p>Remaining stock needed: <?= $remaining_quantity ?></p>
          <?php endif; ?>
      
          <?php if (!empty($issued)): ?>
              <h2>Issued Items</h2>
              <table border="1">
                  <thead>
                      <tr>
                          <th>Order ID</th>
                          <th>Item ID</th>
                          <th>Stock ID</th>
                          <th>Issued Quantity</th>
                          <th>Selling Price</th>
                      </tr>
                  </thead>
                  <tbody>
                      <?php foreach ($issued as $issue): ?>
                          <tr>
                              <td><?= $issue['order_id'] ?></td>
                              <td><?= $issue['item_id'] ?></td>
                              <td><?= $issue['stock_id'] ?></td>
                              <td><?= $issue['issue_qty'] ?></td>
                              <td><?= $issue['selling_price'] ?></td>
                          </tr>
                      <?php endforeach; ?>
                  </tbody>
              </table>
          <?php endif; ?>
      </body>
      </html>

  ```

