var express = require("express");
var app = express();
var bodyParser = require("body-parser");
var cors = require("cors");
var mongoose = require("mongoose");
var config = require("./config");

app.use(cors());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

var User = require("./models/user");
var Exercise = require("./models/exercise");

// Connect to mongoose
mongoose.connect(config.MLAB_URI, {
  useNewUrlParser: true
});
mongoose.set("useCreateIndex", true);
var db = mongoose.connection;

// create new user
app.post("/api/exercise/new-user", function(req, res) {
  var user = req.body;
  User.addUser(user, function(err, user) {
    if (err) {
      res.send("username already taken");
      console.log(err.errmsg);
    }
    res.json(user);
  });
});

// add new exercise to the user
app.post("/api/exercise/add", function(req, res) {
  var exercise = req.body;
  if (!exercise.date) {
    exercise.date = Date.now();
  }

  User.getUserById(exercise.userId, function(err, user) {
    if (err) {
      res.send("unknown_id");
      console.log(err);
    } else {
      Exercise.addExercise(exercise, function(err, exercise) {
        if (err) {
          res.send(err.message);
          console.log(err);
        }
        res.json({
          username: user.username,
          description: exercise.description,
          duration: exercise.duration,
          _id: exercise.userId,
          date: exercise.date
        });
      });
    }
  });
});

// get user list
app.get("/api/exercise/users", function(req, res) {
  User.getUsers(function(err, users) {
    if (err) {
      res.send(err.message);
      console.log(err);
    }
    res.json(users);
  });
});

// get exercises log for the user
app.get("/api/exercise/log", function(req, res) {
  var userId = req.query.userId;
  var from = req.query.from || 0; // year 1970
  var to = req.query.to || Date.parse("Aug 9, 2065"); // distant future

  User.getUserById(userId, function(err, user) {

     var obj = {
      queryObject: { userId, date: { $gt: from, $lt: to } },
      callback: function(err, exercises) {
        if (err) {
          res.send(err.message);
          console.log(err);
        }
        res.json({
          _id: userId,
          username: user.username,
          count: exercises.length,
          log: exercises
        });
      },
      limit: req.query.limit,
      sortingOrder: { date: 1 },
      filterObject: { userId: 0, _id: 0, __v: 0 }
    };

    if (err) {
      res.send("unknown_id");
      console.log(err);
    } else {
      Exercise.getExercisesByQueryObject(obj);
    }
  });
});

app.use(express.static("public"));
app.get("/", (req, res) => {
  res.sendFile(__dirname + "/views/index.html");
});

var listener = app.listen(process.env.PORT || 3000, () => {
  console.log("Your app is listening on port " + listener.address().port);
});
