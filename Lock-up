// SPDX-License-Identifier: MIT
pragma solidity >=0.6.0 <0.8.0;
contract EternalStorage {
    mapping(bytes32 => uint256) internal uintStorage;
    mapping(bytes32 => string) internal stringStorage;
    mapping(bytes32 => address) internal addressStorage;
    mapping(bytes32 => bytes) internal bytesStorage;
    mapping(bytes32 => bool) internal boolStorage;
    mapping(bytes32 => int256) internal intStorage;
}
pragma solidity >=0.6.0 <0.8.0;
abstract contract ReentrancyGuard {
    uint256 private constant _NOT_ENTERED = 1;
    uint256 private constant _ENTERED = 2;
    uint256 private _status;
    constructor () internal {
        _status = _NOT_ENTERED;
    }
    modifier nonReentrant() {
        require(_status != _ENTERED, "ReentrancyGuard: reentrant call");
        _status = _ENTERED;
        _;
        _status = _NOT_ENTERED;
    }
}
library SafeMath {
    function tryAdd(uint256 a, uint256 b) internal pure returns (bool, uint256) {
        uint256 c = a + b;
        if (c < a) return (false, 0);
        return (true, c);
    }
    function trySub(uint256 a, uint256 b) internal pure returns (bool, uint256) {
        if (b > a) return (false, 0);
        return (true, a - b);
    }
    function tryMul(uint256 a, uint256 b) internal pure returns (bool, uint256) {
        if (a == 0) return (true, 0);
        uint256 c = a * b;
        if (c / a != b) return (false, 0);
        return (true, c);
    }
    function tryDiv(uint256 a, uint256 b) internal pure returns (bool, uint256) {
        if (b == 0) return (false, 0);
        return (true, a / b);
    }
    function tryMod(uint256 a, uint256 b) internal pure returns (bool, uint256) {
        if (b == 0) return (false, 0);
        return (true, a % b);
    }
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");
        return c;
    }
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b <= a, "SafeMath: subtraction overflow");
        return a - b;
    }
    function mul(uint256 a, uint256 b) internal pure returns (uint256) {
        if (a == 0) return 0;
        uint256 c = a * b;
        require(c / a == b, "SafeMath: multiplication overflow");
        return c;
    }
    function div(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b > 0, "SafeMath: division by zero");
        return a / b;
    }
    function mod(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b > 0, "SafeMath: modulo by zero");
        return a % b;
    }
    function sub(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b <= a, errorMessage);
        return a - b;
    }
    function div(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b > 0, errorMessage);
        return a / b;
    }
    function mod(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b > 0, errorMessage);
        return a % b;
    }
}
abstract contract Context {
    function _msgSender() internal view virtual returns (address payable) {
        return msg.sender;
    }

    function _msgData() internal view virtual returns (bytes memory) {
        this;
        return msg.data;
    }
}
abstract contract Ownable is Context {
    address private _owner;
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    constructor () internal {
        address msgSender = _msgSender();
        _owner = msgSender;
        emit OwnershipTransferred(address(0), msgSender);
    }
    function owner() public view virtual returns (address) {
        return _owner;
    }
    modifier onlyOwner() {
        require(owner() == _msgSender(), "Ownable: caller is not the owner");
        _;
    }
    function renounceOwnership() public virtual onlyOwner {
        emit OwnershipTransferred(_owner, address(0));
        _owner = address(0);
    }
    function transferOwnership(address newOwner) public virtual onlyOwner {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        emit OwnershipTransferred(_owner, newOwner);
        _owner = newOwner;
    }
}
library TransferHelper {
    function safeApprove(
        address token,
        address to,
        uint256 value
    ) internal {
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(0x095ea7b3, to, value));
        require(
            success && (data.length == 0 || abi.decode(data, (bool))),
            'TransferHelper::safeApprove: approve failed'
        );
    }
    function safeTransfer(
        address token,
        address to,
        uint256 value
    ) internal {
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(0xa9059cbb, to, value));
        require(
            success && (data.length == 0 || abi.decode(data, (bool))),
            'TransferHelper::safeTransfer: transfer failed'
        );
    }
    function safeTransferFrom(
        address token,
        address from,
        address to,
        uint256 value
    ) internal {
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(0x23b872dd, from, to, value));
        require(
            success && (data.length == 0 || abi.decode(data, (bool))),
            'TransferHelper::transferFrom: transferFrom failed'
        );
    }
    function safeTransferBaseToken(address token, address payable to, uint value, bool isERC20) internal {
        if (!isERC20) {
            to.transfer(value);
        } else {
            (bool success, bytes memory data) = token.call(abi.encodeWithSelector(0xa9059cbb, to, value));
            require(success && (data.length == 0 || abi.decode(data, (bool))), 'TransferHelper: TRANSFER_FAILED');
        }
    }
}
interface IERCBurn {
    function burn(uint256 _amount) external;
    function approve(address spender, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external returns (uint256);
    function balanceOf(address account) external view returns (uint256);
}
interface IUniFactory {
    function getPair(address tokenA, address tokenB) external view returns (address);
}
interface IMigrator {
    function migrate(address lpToken, uint256 amount, uint256 unlockDate, address owner) external returns (bool);
}
contract TokenLocker is Ownable, ReentrancyGuard,EternalStorage {
  using SafeMath for uint256;
  struct TokenLock {
    uint256 lockDate;
    uint256 amount; 
    uint256 initialAmount;
    uint256 unlockDate;
    address owner;
  }
  mapping(address => address[]) public lockedTokens;
  mapping(address => address[]) public lockedUser;
  mapping(address =>  mapping (address => TokenLock)) public tokenLocks;
  struct FeeStruct {
    uint256 ethFee;
    uint256 liquidityFee;
  }
  FeeStruct public gFees;
  address payable devaddr;
  address payable lpaddr;
  IMigrator migrator;
  event onDeposit(address lpToken, address user, uint256 amount, uint256 lockDate, uint256 unlockDate);
  event onWithdraw(address lpToken, uint256 amount);
  constructor(address payable _lpaddr) public {
    devaddr = msg.sender;
    lpaddr = _lpaddr;
    gFees.ethFee = 100 ether;
    gFees.liquidityFee = 10; // 1%
  }
    function setDev(address payable _devaddr) public onlyOwner {
    devaddr = _devaddr;
  }
  function setMigrator(IMigrator _migrator) public onlyOwner {
    migrator = _migrator;
  }
  function setFees(uint256 _ethFee, uint256 _liquidityFee) public onlyOwner {
    gFees.ethFee = _ethFee;
    gFees.liquidityFee = _liquidityFee;
  }
  function lockLPToken (address _lpToken, uint256 _amount, uint256 _unlock_date, address payable _withdrawer) external payable nonReentrant {
    require(_unlock_date < 10000000000, 'TIMESTAMP INVALID');
    require(_amount > 0, 'INSUFFICIENT');
    TransferHelper.safeTransferFrom(_lpToken, address(msg.sender), address(this), _amount);
    uint256 ethFee = gFees.ethFee;
    require(msg.value == ethFee, 'FEE NOT MET');
    uint256 devFee = ethFee;
    devaddr.transfer(devFee);
    uint256 liquidityFee = _amount.mul(gFees.liquidityFee).div(1000);
    TransferHelper.safeTransfer(_lpToken, lpaddr, liquidityFee);
    uint256 amountLocked = _amount.sub(liquidityFee);
	if(!boolStorage[keccak256(abi.encodePacked(_lpToken,_withdrawer))]){
        TokenLock memory token_lock;
        token_lock.lockDate = block.timestamp;
        token_lock.amount = amountLocked;
        token_lock.initialAmount = amountLocked;
        token_lock.unlockDate = _unlock_date;
        token_lock.owner = _withdrawer;
        tokenLocks[_lpToken][_withdrawer] = token_lock;
        boolStorage[keccak256(abi.encodePacked(_lpToken,_withdrawer))] = true;
        if(!boolStorage[keccak256(abi.encodePacked(_withdrawer,_lpToken))]){
          lockedTokens[_lpToken].push(_withdrawer);
          lockedUser[_withdrawer].push(_lpToken);
          boolStorage[keccak256(abi.encodePacked(_withdrawer,_lpToken))] = true;
        }
        emit onDeposit(_lpToken, msg.sender, token_lock.amount, token_lock.lockDate, token_lock.unlockDate);
	}else{
	    require(msg.sender == _withdrawer, '_withdrawer no sender');
	    TokenLock storage tokenLock = tokenLocks[_lpToken][_withdrawer];
		tokenLock.amount= tokenLock.amount.add(amountLocked);
		tokenLock.initialAmount= tokenLock.initialAmount.add(amountLocked);
		tokenLock.lockDate = block.timestamp;
		if(_unlock_date > tokenLock.unlockDate){
		    tokenLock.unlockDate = _unlock_date;
		}
		emit onDeposit(_lpToken, msg.sender, tokenLock.amount, tokenLock.lockDate, tokenLock.unlockDate);
	}
  }
  function relock (address _lpToken,  uint256 _unlock_date) external nonReentrant {
    require(_unlock_date < 10000000000, 'TIMESTAMP INVALID');
    TokenLock storage userLock = tokenLocks[_lpToken][msg.sender];
    require(userLock.owner == msg.sender, 'LOCK MISMATCH');
    require(userLock.unlockDate < _unlock_date, 'UNLOCK BEFORE');
    userLock.unlockDate = _unlock_date;
  }
  function withdraw (address _lpToken, uint256 _amount) external nonReentrant {
    require(_amount > 0, 'ZERO WITHDRAWL');
    TokenLock storage userLock = tokenLocks[_lpToken][msg.sender];
    require(userLock.owner == msg.sender, 'LOCK MISMATCH');
    require(userLock.unlockDate < block.timestamp, 'NOT YET');
    userLock.amount = userLock.amount.sub(_amount);
    if (userLock.amount == 0) {
		boolStorage[keccak256(abi.encodePacked(_lpToken,msg.sender))] = false;
    }
    TransferHelper.safeTransfer(_lpToken, msg.sender, _amount);
    emit onWithdraw(_lpToken, _amount);
  }
  function getLockForToken (address _lpToken) external view 
  returns (address[] memory) {
    address[] memory addr_list = lockedTokens[_lpToken];
    return addr_list;
  }
  function getLockForUser (address _user) external view 
  returns (address[] memory) {
    address[] memory addr_list = lockedUser[_user];
    return addr_list;
  }
  function getUserLockForToken (address _user, address _lpToken) external view 
  returns (uint256, uint256, uint256, uint256, address) {
    TokenLock storage tokenLock = tokenLocks[_lpToken][_user];
    return (tokenLock.lockDate, tokenLock.amount, tokenLock.initialAmount, tokenLock.unlockDate, tokenLock.owner);
  }
}
