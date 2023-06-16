const express = require("express");
const bodyParser = require('body-parser');
const programmer = require("./src/database/tables/programmer");

const app = express();

const port = 5000;

app.get("/", (req, res) => {
  res.sendFile('index.html', {root: __dirname});
});

app.get('/syncDatabase', async (req, res) => {
  const database = require('./src/database/db');

  try {
    await database.sync();
    res.send(`Database succefully sync'ed`);
  } catch (error) {
    res.send(error);
  }

});

app.post('/createProgrammer', async (req, res) => {
  try {
    const params = req.body;
    const properties = ['name', 'python', 'java', 'javascript'];
    const check = properties.every((property) => {
      return property in params;
    });

    if(!check) {
      const propStr = properties.join(', ');
      res.send(`All parameters needed to create a programmer must be sent: ${propStr}`);
      return;
    }

    const newProgrammer = await programmer.create({
      name: params.name,
      python: params.python,
      javascript: params.javascript,
      java: params.java
    });

    res.send(newProgrammer);


  }catch(error) {
    res.send(error);
  }
});

app.get('/retrieveProgrammer', async (req, res) => {
  try {
    const params = req.body;

    if('id' in params) {
      const record = await programmer.findByPk(params.id);

      if(record) {
        res.send(record);
      } else {
        res.send('No programmer found using received ID')
      }

      return;
    }

    const records = await programmer.findAll();
    res.send(records);

  }catch(error) {
    res.send(error);
  }
});

app.use(bodyParser.json());

app.listen(port, () => {
  console.log(`Now listening on port ${port}`)
});