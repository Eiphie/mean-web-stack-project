# MEAN web stack
MEAN is a Javascript-based full-stack framework for building dynamic web applications. It consists of four main components: MongoDB, ExpressJS, Angular, and NodeJS.
- Angular (Frontend): Handles the UI and user requests.
- Express + Node (Backend): Provides API endpoints to send/receive data.
- MongoDB (Database): Stores and retrieves application data.

## Project Overview
With the MEAN stack, you can use JavaScript consistently across the entire application: on the client side, server side, and for database interactions.

## Project SetUp
- Update and upgrade ubuntu
```
sudo apt update
```
```
sudo apt upgrade
```

- Add Certificates and install NodeJs
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
```
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```
```
sudo apt install -y nodejs
```

- Install MongoDB
```
sudo apt-get install -y gnupg
```
```
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
  --dearmor
```
```
lsb_release -sc
```
```
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```
```
sudo apt-get update
```
```
sudo apt-get install -y mongodb-org
```

- Restart and enable server
```
sudo systemctl start mongod
```
```
sudo systemctl enable mongod
```

- Cheack server status that it is active
```
sudo systemctl status mongod
```

- Make sure you have npm installed
```
npm -v
```

- Create a folder named ⁠`Book`
```
mkdir Books && cd Books
```

- Initialize the command `npm init` in the Books diretory
```
npm init
```

- Install `body parser` package
```
sudo npm install body-parser
```

- Add `server.js` file
```
sudo vim server.js
```

- Create a basic Express server
```
sudo npm install express mongoose
```

- Create a folder named `Apps` within the `Books`
```
mkdir apps && cd apps
```

- Add `routes.js` file and paste the code snippet below:
```
sudo vim routes.js
```
```
const Book = require('./models/book');
const path = require('path');

module.exports = function (app) {
    app.get('/book', async (req, res) => {
        try {
            const books = await Book.find();
            res.json(books);
        } catch (err) {
            res.status(500).json({ message: 'Error fetching books', error: err.message });
        }
    });

    app.post('/book', async (req, res) => {
        try {
            const book = new Book({
                name: req.body.name,
                isbn: req.body.isbn,
                author: req.body.author,
                pages: req.body.pages
            });
            const savedBook = await book.save();
            res.status(201).json({
                message: 'Successfully added book',
                book: savedBook
            });
        } catch (err) {
            res.status(400).json({ message: 'Error adding book', error: err.message });
        }
    });

    app.delete('/book/:isbn', async (req, res) => {
        try {
            const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
            if (!result) {
                return res.status(404).json({ message: 'Book not found' });
            }
            res.json({
                message: 'Successfully deleted the book',
                book: result
            });
        } catch (err) {
            res.status(500).json({ message: 'Error deleting book', error: err.message });
        }
    });

    app.get('*', (req, res) => {
        res.sendFile(path.join(__dirname, '../public', 'index.html'));
    });
};
```

- Create a folder named `models` within the `Apps`
```
mkdir models && cd model
```

- Add `book.js` file and paste the code snippet below:
```
sudo vim book.js
```
```
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
    name: { type: String, required: true },
    isbn: { type: String, required: true, unique: true, index: true },
    author: { type: String, required: true },
    pages: { type: Number, required: true, min: 1 }
}, {
    timestamps: true
});

module.exports = mongoose.model('Book', bookSchema);
```

#### Change the directory back to `Books` and create a folder named `public` to access the routes with AngularJS.
```
cd ../..
```
```
mkdir public && cd public
```

- Add `scrpit.js` file and paste the code snippet below:
```
sudo vim script.js
```
```
angular.module('myApp', [])
    .controller('myCtrl', function ($scope, $http) {
        function fetchBooks() {
            $http.get('/book')
                .then(function (response) {
                    $scope.books = response.data;
                })
                .catch(function (error) {
                    console.error('Error fetching books:', error);
                });
        }

        fetchBooks();

        $scope.del_book = function (book) {
            $http.delete('/book/' + book.isbn)
                .then(function () {
                    fetchBooks();
                })
                .catch(function (error) {
                    console.error('Error deleting book:', error);
                });
        };

        $scope.add_book = function () {
            const newBook = {
                name: $scope.Name,
                isbn: $scope.Isbn,
                author: $scope.Author,
                pages: $scope.Pages
            };

            $http.post('/book', newBook)
                .then(function () {
                    fetchBooks();
                    // Clear form fields
                    $scope.Name = '';
                    $scope.Isbn = '';
                    $scope.Author = '';
                    $scope.Pages = '';
                })
                .catch(function (error) {
                    console.error('Error adding book:', error);
                });
        };
    });
```

- Add `index.html` file and paste the code snippet below:
```
sudo vim index.html
```
```
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Book Management</title>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }

        table {
            border-collapse: collapse;
            width: 100%;
        }

        th,
        td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }

        th {
            background-color: #f2f2f2;
        }

        input[type="text"],
        input[type="number"] {
            width: 100%;
            padding: 5px;
        }

        button {
            margin-top: 10px;
            padding: 5px 10px;
        }
    </style>
</head>

<body>
    <h1>Book Management</h1>

    <h2>Add New Book</h2>
    <form ng-submit="add_book()">
        <table>
            <tr>
                <td>Name:</td>
                <td><input type="text" ng-model="Name" required></td>
            </tr>
            <tr>
                <td>ISBN:</td>
                <td><input type="text" ng-model="Isbn" required></td>
            </tr>
            <tr>
                <td>Author:</td>
                <td><input type="text" ng-model="Author" required></td>
            </tr>
            <tr>
                <td>Pages:</td>
                <td><input type="number" ng-model="Pages" required></td>
            </tr>
        </table>
        <button type="submit">Add Book</button>
    </form>

    <h2>Book List</h2>
    <table>
        <thead>
            <tr>
                <th>Name</th>
                <th>ISBN</th>
                <th>Author</th>
                <th>Pages</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            <tr ng-repeat="book in books">
                <td>{{book.name}}</td>
                <td>{{book.isbn}}</td>
                <td>{{book.author}}</td>
                <td>{{book.pages}}</td>
                <td><button ng-click="del_book(book)">Delete</button></td>
            </tr>
        </tbody>
    </table>

    <script>
        angular.module('myApp', [])
            .controller('myCtrl', function ($scope, $http) {
                function fetchBooks() {
                    $http.get('/book')
                        .then(function (response) {
                            $scope.books = response.data;
                        })
                        .catch(function (error) {
                            console.error('Error fetching books:', error);
                        });
                }

                fetchBooks();

                $scope.del_book = function (book) {
                    $http.delete('/book/' + book.isbn)
                        .then(function () {
                            fetchBooks();
                        })
                        .catch(function (error) {
                            console.error('Error deleting book:', error);
                        });
                };

                $scope.add_book = function () {
                    const newBook = {
                        name: $scope.Name,
                        isbn: $scope.Isbn,
                        author: $scope.Author,
                        pages: $scope.Pages
                    };

                    $http.post('/book', newBook)
                        .then(function () {
                            fetchBooks();
                            // Clear form fields
                            $scope.Name = '';
                            $scope.Isbn = '';
                            $scope.Author = '';
                            $scope.Pages = '';
                        })
                        .catch(function (error) {
                            console.error('Error adding book:', error);
                        });
                };
            });
    </script>
</body>

</html>
```




