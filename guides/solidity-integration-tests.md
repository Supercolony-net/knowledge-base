# Configure integration tests for Solidity Smart Contracts

To run our Solidity integration tests we are using [OpenZeppelin Test Environment](https://docs.openzeppelin.com/test-environment/0.1/).
The library has certain benefits:
- Agnostic to test runner (e.g. Mocha)
- Relatively fast brings up preconfigured env with default addresses. Each test is being run against fresh config, hence no need in custom tear down
- Allow to configure env

Follow the steps to configure:
1. Install certain dependencies:
```
    npm install --save-dev @openzeppelin/test-environment @openzeppelin/test-helpers chai mocha truffle
```
2. Add script to `package.json`:
```json
{
  "scripts": {
    "clean": "rimraf build",
    "compile": "npm run clean && truffle compile",
    "test": "npm run compile && mocha --exit --recursive"
  }
}
```
3. *Optional*. Create `test-setup.js` and add `--require ./test/tests-setup.js` to `test` script:
```javascript
const chai = require('chai')
const chaiAsPromised = require('chai-as-promised')
chai.use(chaiAsPromised)
require('@openzeppelin/test-helpers') // to support BN (web3 type for big numbers)

global.expect = chai.expect
```
4. *Optional*. Configure test env with `test-environment.config.js` ([more info](https://docs.openzeppelin.com/test-environment/0.1/getting-started#configuration))
4. Run tests with `npm run test` or `npm run test -- -g "Specific test teamplate"`

### FAQ

1. **How test big numbers?** If you add `require('@openzeppelin/test-helpers')`, `BigNumber` will be supported by default. Use chai to assert BN output:
```javascript
expect(await vestingContract.balanceOf(user)).to.be.bignumber.equal('1000') 
```
2. **How to set owner?** Use [test-environment API](https://docs.openzeppelin.com/test-environment/0.1/api):
```javascript
const { accounts, defaultSender, contract } = require('@openzeppelin/test-environment');

const SCOLToken = contract.fromArtifact('SCOLToken')

describe('SCOL Token: Basic', async () => {
    const owner = defaultSender
    const [minter, admin] = accounts
    let contract

    beforeEach(async () => {
        contract = await SCOLToken.new(minter, { from: owner })
    })
    
    it('should be initialized with owner', async () => {
        expect(await contract.owner()).to.equal(owner)
    })
```
3. **How to test Events and Transaction reverts?** Use [test-helpers API](https://docs.openzeppelin.com/test-helpers/0.5/api):
```javascript
const { expectEvent, expectRevert } = require('@openzeppelin/test-helpers');

it('should change owner', async () => {
    const receipt = await contract.transferOwnership(admin)
    expect(await contract.owner()).to.equal(admin)
    expectEvent(receipt, 'OwnershipTransferred', { previousOwner: owner, newOwner: admin })
})
```