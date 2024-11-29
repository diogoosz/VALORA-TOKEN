# Valora Token Contract

This is the Solidity contract for the **Valora Token (VAL)**, following the BEP-20 standard.

## Contract Code

updated version of the code with automatic value conversions (VEXIS, VEX, mVAL, cVAL, dVAL), internally in the contract.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

// Interface of BEP20
interface IBEP20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

contract ValoraToken is IBEP20 {
    string public constant name = "Valora";
    string public constant symbol = "VAL";
    uint8 public constant decimals = 8;

    uint256 private _totalSupply;
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;

    // Constructor that defines the initial supply of tokens
    constructor(uint256 initialSupply) {
        _totalSupply = initialSupply * 10 ** uint256(decimals);  // Adjusted for 8 decimals
        _balances[msg.sender] = _totalSupply;
        emit Transfer(address(0), msg.sender, _totalSupply);
    }

    // Standard BEP20 Functions

    // Returns the total supply of tokens in circulation
    function totalSupply() external view override returns (uint256) {
        return _totalSupply;
    }

    // Returns the balance of tokens for a specific address
    function balanceOf(address account) external view override returns (uint256) {
        return _balances[account];
    }

    // Transfers tokens to a recipient (considering decimals)
    function transfer(address recipient, uint256 amount) external override returns (bool) {
        uint256 amountWithDecimals = amount * (10 ** uint256(decimals));  // Convert to the smallest unit
        
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[msg.sender] >= amountWithDecimals, "ERC20: transfer amount exceeds balance");

        _balances[msg.sender] -= amountWithDecimals;
        _balances[recipient] += amountWithDecimals;

        emit Transfer(msg.sender, recipient, amountWithDecimals);
        return true;
    }

    // Approves a spender to spend tokens on behalf of the owner
    function approve(address spender, uint256 amount) external override returns (bool) {
        uint256 amountWithDecimals = amount * (10 ** uint256(decimals));  // Convert to the smallest unit
        _allowances[msg.sender][spender] = amountWithDecimals;
        emit Approval(msg.sender, spender, amountWithDecimals);
        return true;
    }

    // Transfers tokens from one address to another, with permission (considering decimals)
    function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {
        uint256 amountWithDecimals = amount * (10 ** uint256(decimals));  // Convert to the smallest unit
        
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[sender] >= amountWithDecimals, "ERC20: transfer amount exceeds balance");
        require(_allowances[sender][msg.sender] >= amountWithDecimals, "ERC20: transfer amount exceeds allowance");

        _balances[sender] -= amountWithDecimals;
        _balances[recipient] += amountWithDecimals;
        _allowances[sender][msg.sender] -= amountWithDecimals;

        emit Transfer(sender, recipient, amountWithDecimals);
        return true;
    }

    // Returns the allowance for a spender to spend from an owner's account
    function allowance(address owner, address spender) external view override returns (uint256) {
        return _allowances[owner][spender];
    }

    // Function for distributing tokens to multiple addresses
    function distributeTokens(address[] memory recipients, uint256[] memory amounts) public {
        require(recipients.length == amounts.length, "ERC20: recipients and amounts length mismatch");

        uint256 totalAmount = 0;
        for (uint256 i = 0; i < amounts.length; i++) {
            totalAmount += amounts[i] * (10 ** uint256(decimals));  // Considers decimals when summing
        }

        require(_balances[msg.sender] >= totalAmount, "ERC20: insufficient balance for distribution");

        for (uint256 i = 0; i < recipients.length; i++) {
            uint256 amountWithDecimals = amounts[i] * (10 ** uint256(decimals));  // Convert to the smallest unit
            _balances[msg.sender] -= amountWithDecimals;
            _balances[recipients[i]] += amountWithDecimals;
            emit Transfer(msg.sender, recipients[i], amountWithDecimals);
        }
    }

    // Burn function to remove tokens from circulation, considering decimals
    function burn(uint256 amount) public {
        uint256 amountWithDecimals = amount * (10 ** uint256(decimals));  // Convert to the smallest unit
        
        require(_balances[msg.sender] >= amountWithDecimals, "ERC20: burn amount exceeds balance");
        
        _balances[msg.sender] -= amountWithDecimals;
        _totalSupply -= amountWithDecimals;
        
        emit Transfer(msg.sender, address(0), amountWithDecimals); // Emits a transfer to address 0 (burning)
    }

    // Function to know the smaller unit of Valora
    function getUnitName(uint256 amount) public pure returns (string memory) {
        if (amount == 10 ** 6) {
            return "VEX";
        } else if (amount == 10 ** 7) {
            return "VEXIS";
        } else if (amount == 10 ** 5) {
            return "mVAL";
        } else if (amount == 10 ** 4) {
            return "cVAL";
        } else if (amount == 10 ** 3) {
            return "dVAL";
        } else {
            return "VAL";
        }
    }
}
