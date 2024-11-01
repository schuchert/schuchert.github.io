---
layout: post
draft: false
title: Practicing Katas in Typescript
description: Been practicing Katas in typescript and decided to script basic project creation
category: [ software ]
tags: [ typescipt, kata ]
---

## Background
I've been practicing coding katas in typescript. I found a [good resource for setting up a project here](https://blog.alexrusin.com/setting-up-a-modern-node-js-project-with-typescript-and-jest/).

However, after going through these steps several times, I got a bit tired of doing the manual work. So here's a
bare-bones version of those instructions to get you up and running quickly.


```bash
if [ "$#" -eq 0 ]
then
  echo "Provide Name of Directory as first argument"
  exit 1
fi

NAME=$1

function initGit() {
  git init
  echo '.idea/' >> .gitignore
  echo 'node_modules/' >> .gitignore
  git add .
  git commit -m 'initial project created'
}

function createSrc() {
    mkdir -p src
    cat > 'src/helloWorld.ts' <<- EOF
		export function hello(name: string = 'World'): string {
		  return \`Hello \${name}\`;
		}
	EOF
}

function createTest() {
    mkdir -p src/__tests__
    cat > src/__tests__/helloWorld.tests.ts <<- EOF
		import { hello } from '../helloWorld';

		describe('helloWorld', () => {
		  it('defaults to world', () => {
		    expect(hello()).toEqual('Hello World');
		  });

		  it('returns name provided', () => {
		    expect(hello('Brett')).toEqual('Hello Brett');
		  });
		});
	EOF
}

function updateTestScript() {
  sed -i '' 's/^.*"test".*$/    "test": "jest"/g' package.json
}

function writeJestConfig() {
  cat > jest.config.ts << EOF
		module.exports = {
		  preset: 'ts-jest',
		  testEnvironment: 'node',
		  roots: ['./src'],
 		  transform: {
		    '^.+\\.tsx?$': 'ts-jest',
		  },
		  testRegex: '(/__tests__/.*|(\\.|/)(test|spec))\\.tsx?$',
		  moduleFileExtensions: ['ts', 'tsx', 'js', 'jsx', 'json', 'node'],
		};
	EOF
}

initializeProject() {
  mkdir $NAME
  cd $NAME

  npm init -y
  npm install typescript ts-node @types/node @tsconfig/node20 --save-dev
  npx tsc --init --target es2023
  npm install jest ts-jest @types/jest --save-dev
}

initializeProject
updateTestScript
writeJestConfig
createSrc
createTest
initGit
```
