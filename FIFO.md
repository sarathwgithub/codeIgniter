----------------------------------------------------------------
**To implement the FIFO issuing of items in CodeIgniter 4**
----------------------------------------------------------------

Step 01: Update Stock Model
  ```php
      protected $allowedFields    = ['product_id', 'quantity', 'price', 'date_received','issue_qty', 'created_at', 'updated_at'];
  ```

Step 02: Update Stock Controller
  ```php
      public function updateIssuedQuantity($stock_id, $issue_qty)
      {
          $stockModel = new StockModel();
          $stockModel->where('id', $stock_id)
              ->set('issue_qty', 'issue_qty + ' . $issue_qty, false)
              ->update();
      }
  ```

Step 03 : Add Issue Model
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

Step 04: Update Stock Controller
  ```php
      use App\Models\IssueModel;

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
  
          $stocks = $itemStockModel->getAvailableStock($item_id, $quantity);
          $remainingQuantity = $quantity;
          $issued = [];
  
          foreach ($stocks as $stock) {
              if ($remainingQuantity <= 0) {
                  break;
              }
  
              $availableQty = $stock['qty'] - $stock['issue_qty'];
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
  
          if ($remainingQuantity > 0) {
              return $this->response->setStatusCode(ResponseInterface::HTTP_BAD_REQUEST)
                  ->setJSON(['message' => 'Not enough stock available']);
          }
  
          return $this->response->setJSON($issued);
      }
  ```
Step 05: Create issue table
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

