|提案|标题|作者| 讨论  |状态|类型|创建|
|:----:|:----|:----:|:----|:----|:----:|:----|
|9|通用证书合约标准|ice0077<qhfice@126.com>，|<URL>|草稿|向下兼容|2020-9-26|

## 简介

一种通用证书合约的标准模板

## 摘要

本标准可以被其他合约调用和通用发行的基本功能。 我们考虑了证书拥有者、证书发行方的查询和使用。证书拥有者拥有对数字证书的所有权。证书发行方拥有对数字证书的发行权和解释权。 

以下标准指定了关键字的事件格式，其中包括“ ERC721”，“ Ownable”，“ RoleControl”，“ IERC721Mint”。 为各种用户提供绝对和客观的证书存放记录。用户（包括其他合约）可以调用合约来颁发证书。

## 动机

* 自树图主网上线以来，尚未有对链上发行认证类证书的标准。
* 随着社区学院的培训告一段落，为了能够真实有效的展现学院的工作以及学员学习成果的认证。进行链上证书的发放。同时，将认证证书作为一个标准进行发行，以便于保证后续树图其他生态认证证书发放的规范性、有效性。
* 使用区块链智能合约减少证书颁发机构的证书存储服务器的租赁和维护成本； 减少支付给第三方电子签名公司的年费； 避免由于自然或人为原因而导致证书数据丢失或损坏的风险。
* 证书发行单位可以有效提高所发行证书的权威和共识。
* 用户可以更方便，快捷，公开地查询证书； 避免证书丢失的风险。
## 规范

```plain
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "./RoleControl.sol";
import "./IERC721Mint.sol";
```
* 证书颁发者使用合约“ Ownable.sol”
* 证书拥有者使用合同“ RoleControl.sol”

注意：该标准指定了格式，并且已经有特定的实现方法。

## 基本原理

* 证书发行者在证书信息输入窗口中输入信息，并生成证书
```plain
abstract contract Ownable is Context {
    address private _owner;
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
```
* 证书拥有者可以根据帐户查看和下载证书
```plain
abstract contract RoleControl is Context {
    using EnumerableSet for EnumerableSet.AddressSet;
    using Address for address;
    struct RoleData {
        EnumerableSet.AddressSet members;
        bytes32 adminRole;
    }
```
## 向后兼容

* TBD
## 测试案例

Conflux大学学院毕业证书

* [证书主合约](https://github.com/ice0077/CIP-9/blob/main/Certificate.sol)
```plain
...
contract Certificate is ERC721, Ownable, RoleControl, IERC721Mint {
    bytes32 private constant MINT_ADMIN_ROLE = keccak256("MINTER_ROLE");
    string private constant erc721Name = "College Certificate NFT";
    string private constant erc721Symbol = "CC-NFT";
    uint256 public tokenIdIndex;
    mapping(string => bool) tokenURIMap;
    constructor () public ERC721(erc721Name, erc721Symbol) { 
        _setupRole(MINT_ADMIN_ROLE, msg.sender);
    }
   ...
```

* [证书合约](https://github.com/ice0077/CIP-9/blob/main/ERC721.sol)
```plain
...
contract ERC721 is Context, ERC165, IERC721, IERC721Metadata, IERC721Enumerable {
    using SafeMath for uint256;
    using Address for address;
    using EnumerableSet for EnumerableSet.UintSet;
    using EnumerableMap for EnumerableMap.UintToAddressMap;
    using Strings for uint256;
    bytes4 private constant _ERC721_RECEIVED = 0x150b7a02;
    mapping (address => EnumerableSet.UintSet) private _holderTokens;
    EnumerableMap.UintToAddressMap private _tokenOwners;
    mapping (uint256 => address) private _tokenApprovals;
    mapping (address => mapping (address => bool)) private _operatorApprovals;
    string private _name;
    string private _symbol;
    mapping (uint256 => string) private _tokenURIs;
    string private _baseURI;
  ...
```
* [证书合约](https://github.com/ice0077/CIP-9/blob/main/IERC721Mint.sol)
```plain
// SPDX-License-Identifier: MIT
pragma solidity >=0.6.0 <0.8.0;
interface IERC721Mint {
    function mint(address to, string memory tokenURI) external returns(uint256 tokenId);
    function burn(uint256 tokenId) external;
    event MintEvent(address operator, address to, string tokenURI, uint256 tokenId);
}
```
* [证书创建](https://github.com/ice0077/CIP-9/blob/main/Ownable.sol)
```plain
// SPDX-License-Identifier: MIT
pragma solidity >=0.6.0 <0.8.0;
import "../GSN/Context.sol";
abstract contract Ownable is Context {
    address private _owner;
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    constructor () internal {
        address msgSender = _msgSender();
        _owner = msgSender;
        emit OwnershipTransferred(address(0), msgSender);
    }
    function owner() public view returns (address) {
        return _owner;
    }
    /**
     * @dev Throws if called by any account other than the owner.
     */
    modifier onlyOwner() {
        require(_owner == _msgSender(), "Ownable: caller is not the owner");
        _;
    }
    function renounceOwnership() public virtual onlyOwner {
        emit OwnershipTransferred(_owner, address(0));
        _owner = address(0);
    }
    function transferOwnership(address newOwner) public virtual onlyOwner {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        emit OwnershipTransferred(_owner, newOwner);
        _owner = newOwner;
    }
}
```
* [证书拥有者合约](https://github.com/ice0077/CIP-9/blob/main/RoleControl.sol)
```plain
...
abstract contract RoleControl is Context {
    using EnumerableSet for EnumerableSet.AddressSet;
    using Address for address;
    struct RoleData {
        EnumerableSet.AddressSet members;
        bytes32 adminRole;
    }
    mapping (bytes32 => RoleData) private _roles;
    bytes32 public constant DEFAULT_ADMIN_ROLE = 0x00;
    event RoleAdminChanged(bytes32 indexed role, bytes32 indexed previousAdminRole, bytes32 indexed newAdminRole);
    event RoleGranted(bytes32 indexed role, address indexed account, address indexed sender);
    event RoleRevoked(bytes32 indexed role, address indexed account, address indexed sender);
    function hasRole(bytes32 role, address account) public view returns (bool) {
        return _roles[role].members.contains(account);
    }
    function getRoleMemberCount(bytes32 role) public view returns (uint256) {
        return _roles[role].members.length();
    }
    function getRoleMember(bytes32 role, uint256 index) public view returns (address) {
        return _roles[role].members.at(index);
    }
    function getRoleAdmin(bytes32 role) public view returns (bytes32) {
        return _roles[role].adminRole;
    }
    function renounceRole(bytes32 role, address account) public virtual {
        require(account == _msgSender(), "AccessControl: can only renounce roles for self");
        _revokeRole(role, account);
    }
    
    function _setupRole(bytes32 role, address account) internal virtual {
        _grantRole(role, account);
    }
    
    function _removeRole(bytes32 role, address account) internal virtual {
        _revokeRole(role, account);
    }
    
    function _setRoleAdmin(bytes32 role, bytes32 adminRole) internal virtual {
        emit RoleAdminChanged(role, _roles[role].adminRole, adminRole);
        _roles[role].adminRole = adminRole;
    }
    function _grantRole(bytes32 role, address account) private {
        if (_roles[role].members.add(account)) {
            emit RoleGranted(role, account, _msgSender());
        }
    }
    function _revokeRole(bytes32 role, address account) private {
        if (_roles[role].members.remove(account)) {
            emit RoleRevoked(role, account, _msgSender());
        }
    }
}
```

## 执行

* 标准可以嵌入其他合约中，也可以扩展。
## 安全注意事项

* TBD
## 版权

* 依据[CC0](https://creativecommons.org/publicdomain/zero/1.0/?fileGuid=ioPgblI7nGIBMFSo).放弃版权和相关权


