# Smart contract tutorial
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
