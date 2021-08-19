# esm-cjs-shim

> Note: only needed if node's static analysis fails: https://nodejs.org/dist/latest/docs/api/esm.html#esm_interoperability_with_commonjs

Problem: importing CJS modules from ESM in node means you get the `exports` object as your default export, so you can't do named imports.

Solution: write a `--loader` that shims CJS modules loaded from ESM so they have named exports.

When the ESM loader resolves something to be CommonJS, rewrite the resolution to an ESM shim.

ESM shim does 2x things:
export * from 'esm-shim://export/<encoded payload>';
import 'esm-shim://discover-exports/<encoded payload>';

// //discover-exports/
import cjs from 'commonjs-module';
const allExports = Buffer.from(JSON.stringify(Object.keys(cjs)).toString('base64');
await import(`esm-shim://report-exports/<unique ID>/${ allExports }`);

// //export/
import cjs from 'commonjs-module';
const {foo, bar, baz} = cjs;
export {foo, bar, baz};

For every CJS imported from ESM, there are 3x "virtual" modules created:
esm-shim://export/
esm-shim://discover-exports/
esm-shim://report-exports/
