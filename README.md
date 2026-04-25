# Bank.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/**
 * @title SimpleBank
 * @dev Kullanıcıların Ether yatırıp çekebileceği temel bir banka kontratı.
 */
contract SimpleBank {
    // Kullanıcı adreslerini bakiyeleriyle eşleştirir
    mapping(address => uint256) private balances;

    // Olaylar (Frontend veya takip için log tutar)
    event Deposit(address indexed user, uint256 amount);
    event Withdraw(address indexed user, uint256 amount);

    /**
     * @dev Kontrata Ether yatırılmasını sağlar.
     */
    function deposit() public payable {
        require(msg.value > 0, "Sifirdan buyuk miktar yatirmalisiniz");
        balances[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }

    /**
     * @dev Kullanıcının bakiyesinden Ether çekmesini sağlar.
     * @param _amount Çekilmek istenen miktar (wei cinsinden)
     */
    function withdraw(uint256 _amount) public {
        require(balances[msg.sender] >= _amount, "Yetersiz bakiye");
        
        // Önce bakiyeyi düşürürüz (Reentrancy saldırılarına karşı önlem)
        balances[msg.sender] -= _amount;
        
        // Transfer işlemini gerçekleştir
        (bool success, ) = payable(msg.sender).call{value: _amount}("");
        require(success, "Transfer basarisiz oldu");

        emit Withdraw(msg.sender, _amount);
    }

    /**
     * @dev Sorgulayan kişinin bakiyesini döndürür.
     */
    function getBalance() public view returns (uint256) {
        return balances[msg.sender];
    }

    /**
     * @dev Kontratın toplam içindeki miktarını döndürür.
     */
    function getContractBalance() public view returns (uint256) {
        return address(this).balance;
    }
}
