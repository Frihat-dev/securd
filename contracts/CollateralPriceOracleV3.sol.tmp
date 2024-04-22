// SPDX-License-Identifier: MIT
pragma solidity ^0.8.8;
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
import "./interfaces/IUniswapV2Pair.sol";
import "./interfaces/IAssetPriceOracle.sol";
import "./libraries/ConstantLib.sol";

contract CollateralPriceOracle is AccessControlEnumerable {
    using Math for uint256;
    IAssetPriceOracle public assetPriceOracle;

    event UpdateAssetPriceOracle(address indexed assetPriceOracle_);

    constructor(address admin_, address assetPriceOracle_) {
        require(assetPriceOracle_ != address(0), "Securd: zero address");
        require(admin_ != address(0), "Securd: zero address");
        assetPriceOracle = IAssetPriceOracle(assetPriceOracle_);
        _setupRole(DEFAULT_ADMIN_ROLE, admin_);
    }

    function updateAssetPriceOracle(
        address assetPriceOracle_
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        require(assetPriceOracle_ != address(0), "Securd: zero address");
        assetPriceOracle = IAssetPriceOracle(assetPriceOracle_);
        emit UpdateAssetPriceOracle(assetPriceOracle_);
    }

    function getCollateralPrice(
        address collateral_
    ) public view returns (uint256 amount0_, uint256 amount1_, uint256 price_) {
        require(collateral_ != address(0), "Securd: zero address");

        (amount0_, amount1_, ) = IUniswapV2Pair(collateral_).getReserves();
        uint256 totalSupply_ = IUniswapV2Pair(collateral_).totalSupply();
        uint256 k = amount0_ * amount1_;
        address asset0_ = IUniswapV2Pair(collateral_).token0();
        address asset1_ = IUniswapV2Pair(collateral_).token1();

        uint256 price0_ = assetPriceOracle.getAssetPrice(asset0_);
        uint256 price1_ = assetPriceOracle.getAssetPrice(asset1_);
        amount0_ =
            (Math.sqrt((k * price0_) / price1_) * DECIMAL_FACTOR) /
            totalSupply_;
        amount1_ =
            (Math.sqrt((k * price0_) / price1_) * DECIMAL_FACTOR) /
            totalSupply_;
        price_ = (amount0_ * price0_ + amount1_ * price1_) / DECIMAL_FACTOR;
    }

    function getCollateralFairAmount(
        address collateral_,
        uint256 amount_
    ) public view returns (uint256 amount0_, uint256 amount1_) {
        (amount0_, amount1_, ) = getCollateralPrice(collateral_);
        amount0_ = (amount_ * amount0_) / DECIMAL_FACTOR;
        amount1_ = (amount_ * amount1_) / DECIMAL_FACTOR;
    }

    function getCollateralAmount(
        address collateral_,
        uint256 amount_
    ) public view returns (uint256 amount0_, uint256 amount1_) {
        (amount0_, amount1_, ) = IUniswapV2Pair(collateral_).getReserves();
        uint256 totalSupply_ = IUniswapV2Pair(collateral_).totalSupply();
        amount0_ = (amount_ * amount0_) / totalSupply_;
        amount1_ = (amount_ * amount1_) / totalSupply_;
    }
}
