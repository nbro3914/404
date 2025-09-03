app.post("/login", async (req,res)=>{

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
        { id:user.id, name:user.name, phoneNumber: user.phoneNumber },
        "sage"
    )

    return res.json({
        name: user.name,
        phoneNumber: user.phoneNumber,
        token: token
    })
})