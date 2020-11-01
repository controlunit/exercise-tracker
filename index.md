const md5 = require('crypto-js/md5');
const findOrCreate = require('mongoose-find-or-create')
const shortid = require('shortid');
const express = require('express')
const app = express()
const bodyParser = require('body-parser')
const cors = require('cors')
const mongoose = require('mongoose')
const url = 'mongodb://127.0.0.1:27017/mydb'

mongoose.connect(url, { useNewUrlParser: true })

const db = mongoose.connection

db.once('open', _ => {
  console.log('Database connected:', url)
})

db.on('error', err => {
  console.error('connection error:', err)
})

app.use(cors())

app.use(bodyParser.urlencoded({extended: false}))
app.use(bodyParser.json())
app.use(express.static('public'))
app.get('/', (req, res) => {
  res.sendFile(__dirname + '/public/index.html')
});

// Error Handling middleware
app.use((err, req, res, next) => {
  let errCode, errMessage

  if (err.errors) {
    // mongoose validation error
    errCode = 400 // bad request
    const keys = Object.keys(err.errors)
    // report the first validation error
    errMessage = err.errors[keys[0]].message
  } else {
    // generic or custom error
    errCode = err.status || 500
    errMessage = err.message || 'Internal Server Error'
  }
  res.status(errCode).type('txt')
  .send(errMessage)
})

const listener = app.listen(process.env.PORT || 3000, () => {
  console.log('Your app is listening on port ' + listener.address().port)
})

// The exercise tracker app
// define the database
var userSchema = new mongoose.Schema({
  username: String,
  _id: String,
});
userSchema.plugin(findOrCreate)

// create the Model
var User = mongoose.model('User', userSchema);

app.post("/api/exercise/new-user", (req, res) => {
  var username = req.body.username;
  // create a hash as the user _id
  var _id = md5(username).toString().slice(-7);
  User.findOrCreate({ username: username, _id: _id}, (err, result) => {
    if (err) return console.log(err);
    //console.log("hello")
    //console.log(result)
    res.json({
      username: username,
      _id : _id
    });
  })
});


app.get('/api/exercise/users', (req, res) => {

  User.find(function (err, results) {

    res.json({
      hello: "hello"
    });
    console.log(results)
  });
})

  // Not found middleware
  // must be situated below the post function
  app.use((req, res, next) => {
    return next({status: 404, message: 'not found'})
  })
