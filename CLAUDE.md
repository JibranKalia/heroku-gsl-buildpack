# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Heroku buildpack that installs the GNU Scientific Library (GSL) on Heroku dynos. It's designed to be used with heroku-buildpack-multi to chain multiple buildpacks together.

## Architecture

### Buildpack Structure

Standard Heroku buildpack with three key scripts in `bin/`:

- **detect**: Always returns "GSL" to indicate this buildpack applies
- **compile**: Main script that downloads, extracts, and vendors GSL binaries
- **release**: Empty placeholder (required by buildpack spec)

### Compilation Flow (bin/compile)

The compile script receives two arguments: BUILD_DIR and CACHE_DIR from Heroku.

1. Downloads pre-compiled GSL 1.14 binary from S3: `https://vp-public-dev.s3.us-west-2.amazonaws.com/libraries/gsl-1.14.tar.gz`
2. Extracts to `${BUILD_DIR}/vendor/gsl`
3. Copies to `${HOME}/vendor/gsl` for runtime access
4. Exports environment variables via two mechanisms:
   - `.profile.d/gsl.sh`: Sources at dyno startup
   - `export` file: Chains vars to subsequent buildpacks

Environment variables set: PATH, LD_LIBRARY_PATH, LIBRARY_PATH, CPATH

### Binary Compilation Utility (extra/build_binary)

Ruby script to compile GSL binaries within a Heroku environment:
- Downloads source tarball
- Runs `./configure --prefix /app/vendor/[package]`
- Compiles with `make && make install`
- Uploads compiled binary to S3 using Fog gem
- Requires AWS_KEY and AWS_SECRET environment variables

Usage: `heroku run ./bin/build_binary "http://url/to/source.tar.gz" BUCKET_NAME/output.tar.gz`

## Key Configuration

- Current GSL version: 1.14 (hardcoded in bin/compile:27)
- S3 bucket: vp-public-dev (hardcoded)
- Installation prefix: `/app/vendor/gsl` (Heroku's vendor directory)

## Known Limitations

- Only GSL 1.14 is currently supported (README mentions 1.16 but code uses 1.14)
- S3 bucket location is hardcoded, not configurable via environment variables
- Requires heroku-buildpack-multi fork that supports buildpack chaining via `./export`
