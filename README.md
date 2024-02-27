# Solidity_HomeWork_5-3
# Code dự án Ngân Hàng với Sodidity
 ```solidity
pragma solidity ^0.8.0;

// Contract cha chứa các chức năng cơ bản
contract BaseUserManagement {
    address public admin; // khai báo địa chỉ cho admin và để là public
    mapping(address => bool) public managers;
    mapping(address => bool) public users;

    constructor() {
        admin = msg.sender; // gán người gọi contract là admin, khi ai đó gọi contract thì sẽ mặc định là admin
    }

    modifier onlyAdmin() {
        require(msg.sender == admin, "Only admin can perform this action");
        _;
    }

    modifier onlyManager() {
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

// Contract con kế thừa từ contract cha BaseUserManagement
contract ExtendedUserManagement is BaseUserManagement {
    // Chức năng mới: chỉ admin có thể thêm manager
    function addManager(address _manager) public onlyAdmin {
        super.addManager(_manager); // Gọi hàm từ contract cha
    }

    // Chức năng mới: chỉ admin có thể xóa manager
    function removeManager(address _manager) public onlyAdmin {
        super.removeManager(_manager); // Gọi hàm từ contract cha
    }

    // Chức năng mới: chỉ manager hoặc admin có thể thêm user
    function addUser(address _user) public onlyManager {
        super.addUser(_user); // Gọi hàm từ contract cha
    }

    // Chức năng mới: chỉ manager hoặc admin có thể xóa user
    function removeUser(address _user) public onlyManager {
        super.removeUser(_user); // Gọi hàm từ contract cha
    }

    // Chức năng mới: kiểm tra vai trò của người dùng
    function checkUserRole(address _user) public view returns (string memory) {
        return super.checkUserRole(_user); // Gọi hàm từ contract cha
    }
}

// Hợp đồng xử lý gửi và rút tiền, kế thừa từ hợp đồng quản lý người dùng
contract MoneyManagement is ExtendedUserManagement {
    // Sự kiện khi có tiền được gửi
    event Deposit(address indexed user, uint amount);

    // Hàm gửi tiền
    function deposit() public payable {
        require(users[msg.sender], "Only registered users can deposit");
        emit Deposit(msg.sender, msg.value);
    }

    // Hàm rút tiền
    function withdraw(uint _amount) public {
        require(users[msg.sender], "Only registered users can withdraw");
        require(address(this).balance >= _amount, "Insufficient balance");
        payable(msg.sender).transfer(_amount);
    }
}

```
