const express = require("express")
const mongoose = require("mongoose")
const bcrypt = require("bcryptjs")
const jwt = require("jsonwebtoken")

const app = express()

mongoose.connect("mongodb+srv://nathan:nati216639@cluster0.1lxs6tp.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0")
    .then(() => console.log("mongoDB connected"))

app.use(express.json())

function auth(req, res, next) {
    const token = req.headers["authorization"]
    if (!token) return res.status(401).json({ error: "no token" })

    jwt.verify(token, "sage", (err, decoded) => {
        if (err) return res.status(403).send("unauthorized")
        console.log(decoded)
        req.user = decoded
        next()
    })
}

const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    phoneNumber: { type: Number, unique: true, required: true },
    gender: { type: String },
    password: { type: String },
    balance: { type: Number, default: 0 },
    role: { type: String, default: "customer" }
})

const UserModel = mongoose.model("user", userSchema)

app.post("/createAccount", async (req, res) => {
    const { name, phoneNumber, gender, password, role } = req.body
    const hashedPassword = await bcrypt.hash(password, 8)

    const user = new UserModel({
        name,
        phoneNumber,
        gender,
        password: hashedPassword,
        role
    })

    await user.save()
    res.status(200).json(user)
})

app.post("/login", async (req, res) => {
    const { phoneNumber, password } = req.body

    const user = await UserModel.findOne({ phoneNumber })
    if (!user) {
        return res.status(404).json({ message: "user not found" })
    }

    const isMatch = await bcrypt.compare(password, user.password)
    if (!isMatch) {
        return res.status(404).json({ message: "incorrect phone number or password" })
    }

    user.password = undefined

    const token = jwt.sign(
        { id: user.id, name: user.name, phoneNumber: user.phoneNumber, role: user.role },
        "sage"
    )

    return res.json({
        name: user.name,
        phoneNumber: user.phoneNumber,
        role: user.role,
        token: token
    })
})

app.get("/showBalance", auth, async (req, res) => {
    const user = await UserModel.findById(req.user.id)
    res.send("welcome User " + user.name + "<br/>" + "your account balance is: " + user.balance)
})

app.listen(5800, () => {
    console.log("server started")
})