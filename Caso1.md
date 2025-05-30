# Caso_De_Uso
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

const app = express();
app.use(express.json());

const secret = 'gfr0qwieurnwbycgdsfyae68xbnuiwmickmre09fghvr';

const users = [];

app.post("/register", async(req, res)=>{
    const {email, password} = req.body;
    const passwordCrypt = await bcrypt.hash(password, 8);
    users.push({email, password:passwordCrypt});
    res.status(201).send("Usuário criado com sucesso!");
});

app.post("/login", async(req, res) =>{
    const {email, password} = req.body;
    const user = users.find(verify => verify.email === email);
    if(!user || !(await bcrypt.compare(password, user.password))){
        return res.status(401).send("Credenciais inválidas");
    }
    const token = jwt.sign({email}, secret, {expiresIn: '1h'});
    res.json({token});
});


function authenticateToken(req, res, next){
    const authHeader =  req.headers['authorizaion'];
    const token = authHeader?.split('')[1];
    if(!token) return res.sendStatus(401);

    jwt.verify(token, secret, (err, user)=> {
        if(err) return res.sendStatus(403);
        req.usesr = user;
        next();
    });
}

app.listen(8080, () => console.log("alrighty!"));
