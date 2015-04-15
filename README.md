# mock-local-storage

Mock lodcalStorage for headless unit tests

Inspired by StackOverflow answers and wrapped into rpm package to be used with mocha.

## Motivation

I need to mock out localStorage to run tests of cache implementation with mocha in console (without browser).

## Installation

    npm install mock-local-storage --save-dev

## Usage

    mocha -r mock-local-storage
	
Mocha will require module, which will replace localStorage and sessionStorage on the global object.

There are some caveats with using index operator, though. Browser's localStorage
works with strings and stringifyes objects stored via localStorage[key] notation,
but this implementation does not.

Additionally, test code can provide a callback to be invoked on item insertion.
Mock implementation will invoke it when localStorage.setItem() is called
(but not with localStorage[key] notation).
It can be used to emulate allocation errors, like this:

	describe('test with mock localStorage', () => {
	    afterEach(() => {
	        localStorage.clear();
			// remove callback
	        localStorage.itemInsertionCallback = null;
	    });
	    it('emulate quota exceeded error', () => {
	        localStorage.length.should.equal(0);
			// register callback
	        localStorage.itemInsertionCallback = (len) => {
	            if (len >= 5) {
	                let err = new Error('Mock localStorage quota exceeded');
	                err.code = 22;
	                throw err;
	            }
	        };
	        let handled = false;
	        try {
	            for (let i = 0; i < 10; ++i) {
	                localStorage.setItem(i, i);
	            }
	        } catch (e) {
	            if (e.code == 22) {
	                // handle quota exceeded error
	                handled = true;
	            }
	        }
	        handled.should.be.true;
	        localStorage.length.should.equal(5);
	    });
	});

## Tests

    npm install
    npm test

## License

[MIT](https://github.com/letsrock-today/mock-local-storage/blob/master/LICENSE)
