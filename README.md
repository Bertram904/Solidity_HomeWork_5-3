# Solidity_HomeWork_5-3
# Code dự án Ngân Hàng với Sodidity
 ```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract BaseUserManagement {
   address public admin;
   mapping(address => bool) public managers;
   mapping(address => bool) public users;

   constructor() {
       admin = msg.sender; // gán người gọi contract là admin, khi ai đó gọi contract thì sẽ mặc định là admin
   }

   modifier onlyAdmin {
       require(msg.sender == admin, "Only admin can perform this action");
       _;
   }

   modifier onlyManager {
       require(managers[msg.sender] || msg.sender == admin, "Only manager can perform this action");
       _;
   }

   function addManager(address _manager) public onlyAdmin {
       managers[_manager] = true;
   }

   function removeManager(address _manager) public onlyAdmin {
       managers[_manager] = false;
   }

   function addUser(address _user) public onlyManager {
       users[_user] = true;
   }

   function removeUser(address _user) public onlyManager {
       users[_user] = false;
   }

   function checkUserRole(address _user) public view returns (string memory) {
       if (admin == _user) {
           return "Admin";
       } else if (managers[_user]) {
           return "Manager";
       } else if (users[_user]) {
           return "User";
       } else {
           return "Unknown";
       }
   }
}
contract MoneyManagement is BaseUserManagement {

    mapping(address => uint) public balances; //

// hàm gửi tiền
    function deposit() public payable  {
        balances[msg.sender] += msg.value;
    }

// hàm rút tiền
    function withDraw (uint amount) public {
        require(balances[msg.sender] >= amount, "Cannot withdraw money");
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
}

contract LoanManagement is BaseUserManagement {
    uint private interestRate; // Lãi suất theo năm, đơn vị là phần trăm (ví dụ: 5 tương ứng với 5%)

    // Thiết lập lãi suất
    function setInterestRate(uint _interestRate) public onlyAdmin {
        require(_interestRate >= 0, "Interest rate must be non-negative");
        interestRate = _interestRate;
    }

    // Hàm tính lãi suất cho một khoản vay
    function calculateInterest(uint _principal, uint _months) public view returns (uint) {
        // Tính lãi suất theo tháng
        uint monthlyRate = (_principal * interestRate) / (12 * 100);
        // Tính lãi suất tổng cộng sau _months tháng
        uint totalInterest = monthlyRate * _months;
        return totalInterest;
    }

    // Hàm gửi tiền vay
    function borrow(uint _amount, uint _months) public payable {
        require(users[msg.sender], "Only registered users can borrow");
        require(_amount > 0, "Borrowed amount must be greater than 0");
        
        uint totalInterest = calculateInterest(_amount, _months);
        uint totalPayment = _amount + totalInterest;
        
        require(msg.value >= totalPayment, "Insufficient funds to cover borrowed amount and interest");
        
        // Transfer borrowed amount to user
        payable(msg.sender).transfer(_amount);
        
        // Emit event for borrowing
        emit Borrow(msg.sender, _amount, _months, totalInterest, totalPayment);
    }

    // Sự kiện khi có tiền được vay
    event Borrow(address indexed user, uint amount, uint months, uint interest, uint totalPayment);
}




```
