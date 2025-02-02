# Valora Token Contract Version 1

This is the Solidity contract for the **Valora Token (VAL)**, following the BEP-20 standard.

## Contract Code

Version 1 of the Valora Token with errors

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

// Interface do BEP20
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

    // Construtor que define a quantidade inicial de tokens
    constructor(uint256 initialSupply) {
        _totalSupply = initialSupply * 10 ** uint256(decimals);
        _balances[msg.sender] = _totalSupply;
        emit Transfer(address(0), msg.sender, _totalSupply);
    }

    // Funções padrão do BEP20

    // Retorna o total de tokens em circulação
    function totalSupply() external view override returns (uint256) {
        return _totalSupply;
    }

    // Retorna o saldo de tokens de uma conta específica
    function balanceOf(address account) external view override returns (uint256) {
        return _balances[account];
    }

    // Transfere tokens para um endereço
    function transfer(address recipient, uint256 amount) external override returns (bool) {
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[msg.sender] >= amount, "ERC20: transfer amount exceeds balance");

        _balances[msg.sender] -= amount;
        _balances[recipient] += amount;

        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    // Aprova um endereço para gastar tokens em nome do proprietário
    function approve(address spender, uint256 amount) external override returns (bool) {
        _allowances[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    // Transfere tokens de um endereço para outro, com permissão
    function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[sender] >= amount, "ERC20: transfer amount exceeds balance");
        require(_allowances[sender][msg.sender] >= amount, "ERC20: transfer amount exceeds allowance");

        _balances[sender] -= amount;
        _balances[recipient] += amount;
        _allowances[sender][msg.sender] -= amount;

        emit Transfer(sender, recipient, amount);
        return true;
    }

    // Retorna a quantidade de tokens que um endereço está autorizado a gastar de outro
    function allowance(address owner, address spender) external view override returns (uint256) {
        return _allowances[owner][spender];
    }

    // Função de distribuição de tokens para múltiplos endereços
    function distributeTokens(address[] memory recipients, uint256[] memory amounts) public {
        require(recipients.length == amounts.length, "ERC20: recipients and amounts length mismatch");

        // Verifica se o saldo total do remetente é suficiente para cobrir todas as transferências
        uint256 totalAmount = 0;
        for (uint256 i = 0; i < amounts.length; i++) {
            totalAmount += amounts[i];
        }

        require(_balances[msg.sender] >= totalAmount, "ERC20: insufficient balance for distribution");

        // Realiza a distribuição
        for (uint256 i = 0; i < recipients.length; i++) {
            // Chama a função transfer do próprio contrato usando 'this'
            this.transfer(recipients[i], amounts[i]);
        }
    }

    // Função de queima de tokens (burn)
    function burn(uint256 amount) public {
        require(_balances[msg.sender] >= amount, "ERC20: burn amount exceeds balance");
        
        // Diminui o saldo do remetente
        _balances[msg.sender] -= amount;
        
        // Reduz o totalSupply de tokens
        _totalSupply -= amount;
        
        emit Transfer(msg.sender, address(0), amount); // Emite um evento de transferência para o endereço 0 (queimando)
    }

    // Função para saber o número de unidades menores de VALORA
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
