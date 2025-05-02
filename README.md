# malda-finance-contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MaldaFinance is Ownable {
    IERC20 public stablecoin;
    uint256 public interestRate;
    mapping(address => uint256) public deposits;
    mapping(address => uint256) public depositTimestamps;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount, uint256 interest);

    constructor(address _stablecoin, uint256 _interestRate) {
        stablecoin = IERC20(_stablecoin);
        interestRate = _interestRate;
    }

    function deposit(uint256 amount) external {
        require(amount > 0, "Deposit must be greater than zero");
        stablecoin.transferFrom(msg.sender, address(this), amount);
        deposits[msg.sender] += amount;
        depositTimestamps[msg.sender] = block.timestamp;
        emit Deposited(msg.sender, amount);
    }

    function calculateInterest(address user) public view returns (uint256) {
        uint256 timeElapsed = block.timestamp - depositTimestamps[user];
        return (deposits[user] * interestRate * timeElapsed) / (365 days * 100);
    }

    function withdraw() external {
        require(deposits[msg.sender] > 0, "No funds to withdraw");
        uint256 principal = deposits[msg.sender];
        uint256 interest = calculateInterest(msg.sender);
        deposits[msg.sender] = 0;
        depositTimestamps[msg.sender] = 0;
        stablecoin.transfer(msg.sender, principal + interest);
        emit Withdrawn(msg.sender, principal, interest);
    }
}
1qasw
