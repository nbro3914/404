const isMatch = await bcryipt.compare(password, user.password)
if (!user) return res.status(404).json({ message: "user not found" })
const isMatch = await bcrypt.compare(password, user.password)
if(!isMatch) return res.status(404).json({message:"incorrect password or phone number"})# 404
