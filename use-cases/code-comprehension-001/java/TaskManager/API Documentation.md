## Exercise: API Documentation 
*** enpoint API code ***
/**
 * Product API endpoints
 */
const productRouter = express.Router();

// Get all products with filtering and pagination
productRouter.get('/', async (req, res) => {
  try {
    const {
      category,
      minPrice,
      maxPrice,
      sort = 'createdAt',
      order = 'desc',
      page = 1,
      limit = 20,
      inStock
    } = req.query;

    // Build filter
    const filter = {};

    if (category) {
      filter.category = category;
    }

    if (minPrice !== undefined || maxPrice !== undefined) {
      filter.price = {};
      if (minPrice !== undefined) filter.price.$gte = parseFloat(minPrice);
      if (maxPrice !== undefined) filter.price.$lte = parseFloat(maxPrice);
    }

    if (inStock === 'true') {
      filter.stockQuantity = { $gt: 0 };
    }

    // Calculate pagination
    const skip = (parseInt(page) - 1) * parseInt(limit);

    // Determine sort order
    const sortOptions = {};
    sortOptions[sort] = order === 'asc' ? 1 : -1;

    // Execute query
    const products = await ProductModel.find(filter)
      .sort(sortOptions)
      .skip(skip)
      .limit(parseInt(limit));

    // Get total count for pagination
    const totalProducts = await ProductModel.countDocuments(filter);

    return res.status(200).json({
      products,
      pagination: {
        total: totalProducts,
        page: parseInt(page),
        limit: parseInt(limit),
        pages: Math.ceil(totalProducts / parseInt(limit))
      }
    });
  } catch (error) {
    console.error('Error fetching products:', error);
    return res.status(500).json({
      error: 'Server error',
      message: 'Failed to fetch products'
    });
  }
});

// Get product by ID
productRouter.get('/:productId', async (req, res) => {
  try {
    const { productId } = req.params;

    const product = await ProductModel.findById(productId);

    if (!product) {
      return res.status(404).json({
        error: 'Not found',
        message: 'Product not found'
      });
    }

    return res.status(200).json(product);
  } catch (error) {
    console.error('Error fetching product:', error);

    // Check if error is invalid ObjectId format
    if (error.name === 'CastError') {
      return res.status(400).json({
        error: 'Invalid ID',
        message: 'Invalid product ID format'
      });
    }

    return res.status(500).json({
      error: 'Server error',
      message: 'Failed to fetch product'
    });
  }
});

module.exports = productRouter;

*** Prompt 1: Endpoint Documentation Generation *** 
Product API — Endpoint Documentation

Base path: /products
All responses are JSON.


Table of contents

GET /products
GET /products/:productId
Authentication
Rate limiting and considerations
Error reference


GET /products <a name="get-products"></a>
Description
Returns a paginated list of products. Supports optional filtering by category, price range, and stock availability. Results can be sorted by any product field in ascending or descending order.

Query parameters
ParameterTypeRequiredDefaultDescriptioncategorystringNo—Filter by product category. Exact match. Example: "electronics"minPricenumberNo—Returns products with price ≥ this value.maxPricenumberNo—Returns products with price ≤ this value.inStockbooleanNo—Pass "true" to return only products where stockQuantity > 0.sortstringNo"createdAt"Field to sort results by. Any valid product field e.g. "price", "name".orderstringNo"desc"Sort direction. Accepts "asc" or "desc".pageintegerNo1Page number. Starts at 1.limitintegerNo20Number of results per page.

Response — 200 OK
json{
  "products": [
    {
      "_id": "64a7f3c2e1b2a3d4e5f67890",
      "name": "Wireless Headphones",
      "category": "electronics",
      "price": 79.99,
      "stockQuantity": 42,
      "createdAt": "2024-01-15T08:30:00.000Z"
    }
  ],
  "pagination": {
    "total": 84,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
}
FieldTypeDescriptionproductsarrayProducts matching the filter criteria.pagination.totalintegerTotal matching products across all pages.pagination.pageintegerCurrent page number.pagination.limitintegerResults per page.pagination.pagesintegerTotal number of pages (Math.ceil(total / limit)).

Example 1 — Filter by category, sorted by price
Request
GET /products?category=electronics&sort=price&order=asc&page=1&limit=10
Response — 200 OK
json{
  "products": [
    {
      "_id": "64a7f3c2e1b2a3d4e5f67891",
      "name": "USB-C Cable",
      "category": "electronics",
      "price": 12.99,
      "stockQuantity": 200,
      "createdAt": "2024-02-01T10:00:00.000Z"
    }
  ],
  "pagination": {
    "total": 53,
    "page": 1,
    "limit": 10,
    "pages": 6
  }
}

Example 2 — Filter by price range, in stock only
Request
GET /products?minPrice=50&maxPrice=200&inStock=true
Response — 200 OK
json{
  "products": [
    {
      "_id": "64a7f3c2e1b2a3d4e5f67893",
      "name": "Bluetooth Speaker",
      "category": "electronics",
      "price": 149.99,
      "stockQuantity": 17,
      "createdAt": "2024-01-20T09:00:00.000Z"
    }
  ],
  "pagination": {
    "total": 1,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}

Error responses
StatusErrorMessageCause500"Server error""Failed to fetch products"Unexpected database or server failure.

GET /products/:productId <a name="get-products-id"></a>
Description
Returns a single product by its unique MongoDB ObjectId. Returns 404 if no product exists with that ID and 400 if the ID format is invalid.

Path parameters
ParameterTypeRequiredDescriptionproductIdstringYesA valid MongoDB ObjectId (24-character hex string). Example: "64a7f3c2e1b2a3d4e5f67890"

Response — 200 OK
json{
  "_id": "64a7f3c2e1b2a3d4e5f67890",
  "name": "Wireless Headphones",
  "category": "electronics",
  "price": 79.99,
  "stockQuantity": 42,
  "createdAt": "2024-01-15T08:30:00.000Z",
  "updatedAt": "2024-03-10T11:45:00.000Z"
}

Example 1 — Valid product ID
Request
GET /products/64a7f3c2e1b2a3d4e5f67890
Response — 200 OK
json{
  "_id": "64a7f3c2e1b2a3d4e5f67890",
  "name": "Wireless Headphones",
  "category": "electronics",
  "price": 79.99,
  "stockQuantity": 42,
  "createdAt": "2024-01-15T08:30:00.000Z",
  "updatedAt": "2024-03-10T11:45:00.000Z"
}

Example 2 — Product not found
Request
GET /products/64a7f3c2e1b2a3d4e5f00000
Response — 404 Not Found
json{
  "error": "Not found",
  "message": "Product not found"
}

Example 3 — Invalid ID format
Request
GET /products/not-a-valid-id
Response — 400 Bad Request
json{
  "error": "Invalid ID",
  "message": "Invalid product ID format"
}

Error responses
StatusErrorMessageCause400"Invalid ID""Invalid product ID format"productId is not a valid MongoDB ObjectId.404"Not found""Product not found"No product exists with the given productId.500"Server error""Failed to fetch product"Unexpected database or server failure.

Authentication <a name="authentication"></a>
Neither endpoint currently implements authentication. Both routes are publicly accessible as written. Before deploying to production, consider adding JWT middleware or API key validation depending on your use case.

Rate limiting and special considerations <a name="rate-limiting"></a>
No rate limiting is applied to these endpoints. Three things in the current implementation are also worth addressing before going live:
sort is not validated — any string can be passed, including fields that don't exist on the document. MongoDB will silently return unexpected results. Whitelist the allowed sort fields.
limit has no maximum — a caller can pass ?limit=999999 and pull the entire collection in one request. Enforce a cap with Math.min(parseInt(limit), 100).
page and limit have no type validation — passing ?limit=abc results in NaN. Add input validation before using these values in the query.

Error reference <a name="error-reference"></a>
Status codeWhen it occurs200 OKRequest succeeded.400 Bad RequestInvalid productId format (GET /:productId only).404 Not FoundNo product found with the given productId.500 Internal Server ErrorUnhandled database or server exception.

*** Prompt 2: API Reference Conversion *** 

openapi: 3.0.3
info:
  title: Product API
  description: >
    RESTful API for managing and querying products. Supports filtering by
    category, price range, and stock availability, as well as sorting and
    pagination.
  version: 1.0.0

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: http://localhost:3000/v1
    description: Local development server

tags:
  - name: Products
    description: Endpoints for product retrieval and management

paths:
  /products:
    get:
      tags:
        - Products
      summary: List all products
      description: >
        Returns a paginated list of products. Supports filtering by category,
        price range, and stock status. Results can be sorted by any product
        field in ascending or descending order.
      operationId: getProducts
      parameters:
        - name: category
          in: query
          description: Filter products by category name.
          required: false
          schema:
            type: string
            example: electronics
        - name: minPrice
          in: query
          description: Filter products with price greater than or equal to this value.
          required: false
          schema:
            type: number
            format: float
            minimum: 0
            example: 10.00
        - name: maxPrice
          in: query
          description: Filter products with price less than or equal to this value.
          required: false
          schema:
            type: number
            format: float
            minimum: 0
            example: 500.00
        - name: inStock
          in: query
          description: When set to `true`, returns only products with stock quantity greater than 0.
          required: false
          schema:
            type: string
            enum: [true, false]
            example: true
        - name: sort
          in: query
          description: Field name to sort results by.
          required: false
          schema:
            type: string
            default: createdAt
            example: price
        - name: order
          in: query
          description: Sort direction.
          required: false
          schema:
            type: string
            enum: [asc, desc]
            default: desc
            example: asc
        - name: page
          in: query
          description: Page number for pagination (1-indexed).
          required: false
          schema:
            type: integer
            minimum: 1
            default: 1
            example: 1
        - name: limit
          in: query
          description: Number of products to return per page.
          required: false
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
            example: 20
      responses:
        "200":
          description: A paginated list of products matching the given filters.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ProductListResponse"
              example:
                products:
                  - id: "64b8f3a2c9e77b001f4e3d12"
                    name: "Wireless Headphones"
                    category: "electronics"
                    price: 79.99
                    stockQuantity: 150
                    createdAt: "2024-01-15T10:30:00Z"
                    updatedAt: "2024-03-01T08:00:00Z"
                  - id: "64b8f3a2c9e77b001f4e3d13"
                    name: "Mechanical Keyboard"
                    category: "electronics"
                    price: 129.99
                    stockQuantity: 42
                    createdAt: "2024-02-20T14:45:00Z"
                    updatedAt: "2024-03-05T11:30:00Z"
                pagination:
                  total: 84
                  page: 1
                  limit: 20
                  pages: 5
        "500":
          description: Internal server error.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ServerError"
              example:
                error: "Server error"
                message: "Failed to fetch products"

  /products/{productId}:
    get:
      tags:
        - Products
      summary: Get a product by ID
      description: >
        Returns a single product identified by its unique MongoDB ObjectId.
        Returns 404 if no product exists with the given ID, or 400 if the
        ID format is invalid.
      operationId: getProductById
      parameters:
        - name: productId
          in: path
          description: The unique MongoDB ObjectId of the product.
          required: true
          schema:
            type: string
            pattern: "^[a-fA-F0-9]{24}$"
            example: "64b8f3a2c9e77b001f4e3d12"
      responses:
        "200":
          description: The product matching the given ID.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Product"
              example:
                id: "64b8f3a2c9e77b001f4e3d12"
                name: "Wireless Headphones"
                category: "electronics"
                price: 79.99
                stockQuantity: 150
                createdAt: "2024-01-15T10:30:00Z"
                updatedAt: "2024-03-01T08:00:00Z"
        "400":
          description: The provided product ID is not a valid MongoDB ObjectId.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/BadRequestError"
              example:
                error: "Invalid ID"
                message: "Invalid product ID format"
        "404":
          description: No product was found with the given ID.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/NotFoundError"
              example:
                error: "Not found"
                message: "Product not found"
        "500":
          description: Internal server error.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ServerError"
              example:
                error: "Server error"
                message: "Failed to fetch product"

components:
  schemas:

    Product:
      type: object
      description: Represents a single product in the catalog.
      required:
        - id
        - name
        - category
        - price
        - stockQuantity
        - createdAt
        - updatedAt
      properties:
        id:
          type: string
          description: Unique MongoDB ObjectId for the product.
          pattern: "^[a-fA-F0-9]{24}$"
          example: "64b8f3a2c9e77b001f4e3d12"
        name:
          type: string
          description: Display name of the product.
          example: "Wireless Headphones"
        category:
          type: string
          description: Category the product belongs to.
          example: "electronics"
        price:
          type: number
          format: float
          description: Price of the product in the store's default currency.
          minimum: 0
          example: 79.99
        stockQuantity:
          type: integer
          description: Number of units currently in stock.
          minimum: 0
          example: 150
        createdAt:
          type: string
          format: date-time
          description: ISO 8601 timestamp of when the product was created.
          example: "2024-01-15T10:30:00Z"
        updatedAt:
          type: string
          format: date-time
          description: ISO 8601 timestamp of when the product was last updated.
          example: "2024-03-01T08:00:00Z"

    Pagination:
      type: object
      description: Pagination metadata included in list responses.
      required:
        - total
        - page
        - limit
        - pages
      properties:
        total:
          type: integer
          description: Total number of products matching the applied filters.
          example: 84
        page:
          type: integer
          description: Current page number (1-indexed).
          example: 1
        limit:
          type: integer
          description: Maximum number of products returned per page.
          example: 20
        pages:
          type: integer
          description: Total number of pages available given the current limit.
          example: 5

    ProductListResponse:
      type: object
      description: Response envelope for the list products endpoint.
      required:
        - products
        - pagination
      properties:
        products:
          type: array
          description: Array of products on the current page.
          items:
            $ref: "#/components/schemas/Product"
        pagination:
          $ref: "#/components/schemas/Pagination"

    NotFoundError:
      type: object
      description: Returned when a requested resource does not exist.
      required:
        - error
        - message
      properties:
        error:
          type: string
          enum: ["Not found"]
          example: "Not found"
        message:
          type: string
          example: "Product not found"

    BadRequestError:
      type: object
      description: Returned when the request contains invalid input (e.g. malformed ID).
      required:
        - error
        - message
      properties:
        error:
          type: string
          enum: ["Invalid ID"]
          example: "Invalid ID"
        message:
          type: string
          example: "Invalid product ID format"

    ServerError:
      type: object
      description: Returned when an unexpected server-side error occurs.
      required:
        - error
        - message
      properties:
        error:
          type: string
          enum: ["Server error"]
          example: "Server error"
        message:
          type: string
          example: "Failed to fetch products"


*** Prompt 3: Documentation Style Conversion *** 
Product API Developer GuideWelcome to the Product API integration guide. This document provides the technical details necessary to interact with our product catalog.1. AuthenticationCurrently, the Product API endpoints are publicly accessible. You do not need an API key or JWT token to make requests at this time.Note: Security middleware (such as API Keys or OAuth2) may be implemented in future versions. Keep an eye on this section for updates regarding production deployment requirements.2. Formatting RequestsThe API follows RESTful principles. All requests are made via GET to the base path /products.Query Parameters (GET /products)To refine your search, you can append the following query parameters to the URL:Filtering: Use category, minPrice, maxPrice, and inStock (boolean).Pagination: Use page (starts at 1) and limit (results per row).Sorting: Use sort (field name) and order (asc or desc).Path Parameters (GET /products/:productId)To retrieve a specific item, append the 24-character hex string (MongoDB ObjectId) to the base path.Example: /products/64a7f3c2e1b2a3d4e5f678903. Interpreting ResponsesAll responses are returned in JSON format.Paginated ListWhen fetching multiple products, the response includes a products array and a pagination object to help you manage UI states or subsequent requests.JSON{
  "products": [...],
  "pagination": {
    "total": 84,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
}
Single ProductFetching by ID returns the specific object directly. Fields include _id, name, category, price, stockQuantity, createdAt, and updatedAt.4. Error HandlingThe API uses standard HTTP status codes. Your application logic should check the status code before parsing the body.StatusError MessageCause400Invalid product ID formatThe productId provided is not a valid 24-character hex string.404Product not foundNo product exists with that specific ID.500Server errorAn unexpected failure occurred on the server or database.5. Java Implementation ExampleBelow is a robust example using Java's HttpClient (available in Java 11+) to fetch a list of electronics sorted by price.Javaimport java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;

public class ProductApiClient {

    private static final String BASE_URL = "http://your-api-domain.com/products";

    public static void main(String[] args) {
        // Example: Get electronics, sorted by price ascending
        String queryParams = "?category=electronics&sort=price&order=asc&limit=10";
        
        HttpClient client = HttpClient.newBuilder()
                .connectTimeout(Duration.ofSeconds(10))
                .build();

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(BASE_URL + queryParams))
                .header("Accept", "application/json")
                .GET()
                .build();

        try {
            HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

            if (response.statusCode() == 200) {
                System.out.println("Success! Response:");
                System.out.println(response.body());
            } else {
                System.err.println("Error Status: " + response.statusCode());
                System.err.println("Body: " + response.body());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
Development ConsiderationsValidation: Currently, the API does not validate the sort field. Ensure your application only passes valid product fields to avoid unexpected sorting behavior.Limits: While the API doesn't currently cap the limit parameter, it is best practice to keep this value under 100 for optimal performance.