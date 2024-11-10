**/utils/screenshot.js**


const path = require('path');
const { allure } = require('allure-playwright');

/**
 * Captures a screenshot and attaches it to the Allure report.
 * @param {object} page - Playwright Page object.
 * @param {string} stepName - The name of the step for identification.
 */
async function captureScreenshot(page, stepName) {
  const timestamp = new Date().toISOString().replace(/[^\w\s]/gi, '_');
  const screenshotPath = path.join(__dirname, `../allure-results/screenshots/${stepName}_${timestamp}.png`);

  // Capture the screenshot
  await page.screenshot({ path: screenshotPath, fullPage: true });

  // Attach screenshot to the Allure report
  allure.attachment(stepName, screenshotPath, 'image/png');
}

module.exports = {
  captureScreenshot,
};




-----------------------------------------

**/support/hooks.js**


const { Before, After, AfterAll, AfterStep, setDefaultTimeout } = require('@cucumber/cucumber');
const { captureScreenshot } = require('../utils/screenshot');

// Extend default timeout to handle slow tests
setDefaultTimeout(60000); // 1 minute

Before(async function () {
  // Code to run before each scenario, like setting up the browser, if needed
  console.log('Starting the test scenario...');
});

AfterStep(async function (step) {
  const page = this.page;

  // Ensure `page` exists before capturing a screenshot
  if (page) {
    // Capture a screenshot after every step
    const stepName = step.keyword + step.text;  // Combine step keyword and text for a descriptive name
    await captureScreenshot(page, stepName);
  }
});

After(async function (scenario) {
  // You can add any cleanup here after each scenario (e.g., closing the browser)
  if (scenario.result.status === 'failed') {
    console.log(`Scenario failed: ${scenario.name}`);
  }
});

AfterAll(async function () {
  // Code to run after all tests, like closing the browser, if needed
  console.log('All tests have completed.');
});




------------------------------------


**cucumber.js**

{
  "default": {
    "require": ["./step_definitions/**/*.js", "./support/hooks.js"],
    "format": ["json:./allure-results/cucumber_report.json"],
    "publishQuiet": true
  }
}



----------------------------





![image](https://github.com/user-attachments/assets/15976c2c-aaad-42ea-98da-e6b5e2837d17)





----------------------------------------------------------------------------------------------------------------









Few commands 

# Clean the previous report data
rm -rf allure-results/*

# Run the tests again
npx cucumber-js

# Generate the Allure report
allure generate allure-results --clean

# Open the Allure report in your default browser
allure open
