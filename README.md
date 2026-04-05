# Dutch-Auction
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "@openzeppelin/contracts/access/Ownable.sol";

contract DutchAuction is Ownable {
    uint256 public startPrice;
    uint256 public endPrice;
    uint256 public startTime;
    uint256 public duration;
    uint256 public totalSupply;
    uint256 public sold;

    constructor(uint256 _startPrice, uint256 _endPrice, uint256 _duration, uint256 _totalSupply) {
        startPrice = _startPrice;
        endPrice = _endPrice;
        duration = _duration;
        totalSupply = _totalSupply;
        startTime = block.timestamp;
    }

    function getCurrentPrice() public view returns (uint256) {
        if (block.timestamp >= startTime + duration) return endPrice;
        uint256 elapsed = block.timestamp - startTime;
        return startPrice - ((startPrice - endPrice) * elapsed) / duration;
    }

    function buy(uint256 quantity) external payable {
        uint256 price = getCurrentPrice();
        uint256 totalCost = price * quantity;
        require(msg.value >= totalCost, "Insufficient ETH");
        require(sold + quantity <= totalSupply, "Not enough left");

        sold += quantity;
        // Mint NFTs here
        if (msg.value > totalCost) {
            payable(msg.sender).transfer(msg.value - totalCost);
        }
    }
}
