# Waykichain Smart Contract Tutorial
This is a Waykichain smart contract tutorial 

This is basic template that all smart contracts will have.
```lua
mylib = require "mylib"
-- Must have this above

----Entry function of the smart contract
Main = function()

end
Main()
```
<br>

Lets add some functionality.

We need to add Contract API code from Waykichain Lua library [documentation]( https://wiccdev-webui.readthedocs.io/en/latest/).

Allows us to call data from our blockchain Database.
```lua

GetContractTxParam = function (startIndex, length)
    assert(startIndex > 0, "GetContractTxParam start error(<=0).")
    assert(length > 0, "GetContractTxParam length error(<=0).")
    assert(startIndex+length-1 <= #contract, "GetContractTxParam length ".. length .." exceeds limit: " .. #contract)

    local newTbl = {}
    local i = 1
    for i = 1,length do
      newTbl[i] = contract[startIndex+i-1]
    end
    return newTbl
  end
```
<br>

So now we have our first function. 
Should look like this. 
```lua
mylib = require "mylib"

GetContractTxParam = function (startIndex, length)
    assert(startIndex > 0, "GetContractTxParam start error(<=0).")
    assert(length > 0, "GetContractTxParam length error(<=0).")
    assert(startIndex+length-1 <= #contract, "GetContractTxParam length ".. length .." exceeds limit: " .. #contract)

    local newTbl = {}
    local i = 1
    for i = 1,length do
      newTbl[i] = contract[startIndex+i-1]
    end
    return newTbl
  end
  
  Main = function()
  
  end
  Main()
```
<br>

Next we add a function for adding data to the blockchain DB. [Documentation](https://wiccdev-webui.readthedocs.io/en/latest/Contract/api_debug/)
```lua

WriteStrkeyValueToDb = function (Strkey,ValueTbl)
    local t = type(ValueTbl)
    assert(t == "table","the type of Value isn't table.")

    local writeTbl = {
        key = Strkey,
        length = #ValueTbl,
        value = {}
    }
    writeTbl.value = ValueTbl
    if not mylib.WriteData(writeTbl) then  error("WriteData error") end
end
```
<br>

We are using two Contract API's now.
1. GetContractTxParam - Call blockchain DB
2. WriteStrkeyValueToDb - Save to blockchain DB

```lua

mylib = require "mylib"

GetContractTxParam = function (startIndex, length)
    assert(startIndex > 0, "GetContractTxParam start error(<=0).")
    assert(length > 0, "GetContractTxParam length error(<=0).")
    assert(startIndex+length-1 <= #contract, "GetContractTxParam length ".. length .." exceeds limit: " .. #contract)

    local newTbl = {}
    local i = 1
    for i = 1,length do
      newTbl[i] = contract[startIndex+i-1]
    end
    return newTbl
  end
  
WriteStrkeyValueToDb = function (Strkey,ValueTbl)
    local t = type(ValueTbl)
    assert(t == "table","the type of Value isn't table.")

    local writeTbl = {
        key = Strkey,
        length = #ValueTbl,
        value = {}
    }
    writeTbl.value = ValueTbl
    if not mylib.WriteData(writeTbl) then  error("WriteData error") end
end

 Main = function()
  
  end
  Main()
  
```

<br>


Next we need to add the Serialize Contract API function.
This will serialize our input data. All data needs to be in the hex format on the front end. That means we need to convert strings to hex when a user inputs something.
```lua

--table to String
Serialize = function(obj, hex)
    local lua = ""
    local t = type(obj)

    if t == "table" then
        for i=1, #obj do
            if hex == false then
                lua = lua .. string.format("%c",obj[i])
            elseif hex == true then
                lua = lua .. string.format("%02x",obj[i])
            else
                error("index type error.")
            end
        end
    elseif t == "nil" then
        return nil
    else
        error("can not Serialize a " .. t .. " type.")
    end

    return lua
end
```
<br>

We also need a function to unpack the tables. This is also part of the contract API.  
```lua

Unpack = function (t,i)
    i = i or 1
    if t[i] then
        return t[i], Unpack(t,i+1)
    end
end

```
<br>

Here is our main function. Lets go through this line by line.
```lua
Main = function()
    local key_lenTbl = GetContractTxParam(1 ,4)
    local key_len = mylib.ByteToInteger(Unpack(key_lenTbl))
    local keyTbl = GetContractTxParam(4 +1,key_len)
    local value_lenTbl = GetContractTxParam(4 + key_len + 1 ,4)
    local value_len = mylib.ByteToInteger(Unpack(value_lenTbl))
    local valueTbl = GetContractTxParam(4+key_len+ 4 + 1,value_len)
    local keyStr = Serialize(keyTbl,false)
    WriteStrkeyValueToDb(keyStr, valueTbl)
end
Main()

```
<br>

We created a variable named key_lenTbl (key length table) and declared it to GetContractTxParam(startIndex = 1, length = 4). Basically key is part of a key pair for example. {key_name: value_name} or {firstname: lastname}
```lua
local key_lenTbl = GetContractTxParam(1, 4)
```
<br>

We created a variable named key_len (key length) and declared to mylib.ByteToInterger() which is part of Waykichains Contract API. The naming is straight forward: Byte to Integer. Then we call the Unpack table function and input key length
```lua
local key_len = mylib.ByteToInteger(Unpack(key_lenTbl))
```
<br>

This is for the key table (keyTbl)
```lua
  local keyTbl = GetContractTxParam(4 +1, key_len)
```
<br>

We repeat the same exact process for the value. 

```lua

 local value_lenTbl = GetContractTxParam(4 + key_len + 1 ,4)
 local value_len = mylib.ByteToInteger(Unpack(value_lenTbl))
 local valueTbl = GetContractTxParam(4+key_len+ 4 + 1,value_len)

```
<br>

We call our Serialize function and set keyTbl as a parameter. 
Below that we write to the database by calling our contract API function WriteStrkeyValueToDb then plugging in our key and value

```lua

local keyStr = Serialize(keyTbl, false)
WriteStrkeyValueToDb(keyStr, valueTbl)
```
<br>

Now here is the entire contract in full. 

```lua

mylib = require "mylib"

--Write data into the blockChain
WriteStrkeyValueToDb = function (Strkey,ValueTbl)
    local t = type(ValueTbl)
    assert(t == "table","the type of Value isn't table.")

    local writeTbl = {
        key = Strkey,
        length = #ValueTbl,
        value = {}
    }
    writeTbl.value = ValueTbl
    if not mylib.WriteData(writeTbl) then  error("WriteData error") end
end

--get external call context
GetContractTxParam = function (startIndex, length)
    assert(startIndex > 0, "GetContractTxParam start error(<=0).")
    assert(length > 0, "GetContractTxParam length error(<=0).")
    assert(startIndex+length-1 <= #contract, "GetContractTxParam length ".. length .." exceeds limit: " .. #contract)
    local newTbl = {}
    --local i = 1
    for i = 1,length do
        newTbl[i] = contract[startIndex+i-1]
    end
    return newTbl
end

Serialize = function(obj, hex)
    local lua = ""
    local t = type(obj)

    if t == "table" then
        for i=1, #obj do
            if hex == false then
                lua = lua .. string.format("%c",obj[i])
            elseif hex == true then
                lua = lua .. string.format("%02x",obj[i])
            else
                error("index type error.")
            end
        end
    elseif t == "nil" then
        return nil
    else
        error("can not Serialize a " .. t .. " type.")
    end

    return lua
end

Unpack = function (t,i)
    i = i or 1
    if t[i] then
        return t[i], Unpack(t,i+1)
    end
end

----Entry function of smart contract
Main = function()
    -- cant save a string directly to the blockchain it must be converted to hex
    local key_lenTbl = GetContractTxParam(1 ,4)
    local key_len = mylib.ByteToInteger(Unpack(key_lenTbl))
    local keyTbl = GetContractTxParam(4 +1,key_len)
    local value_lenTbl = GetContractTxParam(4 + key_len + 1 ,4)
    local value_len = mylib.ByteToInteger(Unpack(value_lenTbl))
    local valueTbl = GetContractTxParam(4+key_len+ 4 + 1,value_len)
    local keyStr = Serialize(keyTbl,false)
    WriteStrkeyValueToDb(keyStr, valueTbl)
end
Main()


```
