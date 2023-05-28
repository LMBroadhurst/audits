# Medium Severity Finding: FootiumClub.sol => safeMint

## Vulnerability Detail
Whilst it is called safeMint, the function does not mint safely and actually implements the not recommended
ERC721Upgradable _mint function. Therefore it is possible to mint multiple tokens with the same tokenId.
_safeMint also performs additional checks to ensure that the target address is a contract that implements the
ERC721Receiver interface, which allows it to receive tokens. This helps prevent accidental token loss due to
incorrect addresses or contracts that do not support NFTs.

This function can only be called by the MINTER_ROLE so the risk is much lower vs. a publicly/externally
callable function but is still susceptible to an admin mistake.

## Impact
If multiple NFTs were to be minted with the same tokenId, there would be confusion in the user base and potential
problems when playing the game, as multiple football clubs would have the same tokenIds.

## Code Snippet
The offending code snippet is shown below.

```solidity
function safeMint(address to, uint256 tokenId)
    external
    onlyRole(MINTER_ROLE)
    nonReentrant
    whenNotPaused
    {
        FootiumEscrow escrow = new FootiumEscrow(address(this), tokenId);
        clubToEscrow[tokenId] = escrow;

        // @audit -- Using _mint() instead of _safeMint()
        _mint(to, tokenId);

        emit EscrowDeployed(tokenId, escrow);
    }
```
## Tool used
Manual Review

## Recommendation
Simply changing to the recommended, safer function of _safeMint over _mint.

```solidity
function safeMint(address to, uint256 tokenId)
    external
    onlyRole(MINTER_ROLE)
    nonReentrant
    whenNotPaused
    {
        FootiumEscrow escrow = new FootiumEscrow(address(this), tokenId);
        clubToEscrow[tokenId] = escrow;

        _safeMint(to, tokenId);

        emit EscrowDeployed(tokenId, escrow);
    }
```
