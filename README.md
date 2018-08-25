# GESI

Google Sheets add-on for interacting with EVE ESI API.

## Setup:

1. Install the add-on [HERE](https://chrome.google.com/webstore/detail/gesi/haaceilfjgofjglobglglnafgnjbekoc?utm_source=permalink).
2. Give the script access to what it needs.
3. There will now be a GESI option under the `Add-Ons` option in the menu bar.  Click it and then click `GESI => Authorize Characters`.
4. Click the EVE SSO button in the modal.  Login => select what character you want to authorize => Authorize.
5. Close the modal.
6. (Optional) Repeat step 4 to authorize other characters.
7. Done.

### Script Editor

By default, one does not have access to GESI functions for use in custom functions in the script editor.  In order to gain access to these functions for custom logic, add GESI as a library to your script:

1. Install the add-on, following the instructions above.
2. Within the script editor, click `Resources => Lbraries...` 
3. At the bottom paste `MKpdmT9YX4m_dA5qB8ReTppeSVVadBdJf` into the `Add a library` box and click `Add`.
4. Select the latest version from the dropdown, and click `Save`.

In order to use this, functions must be prepended with `GESI`, which maps to the `Identifier` field in the Libraries modal.  For example, `GESI.universe_types();`

**NOTE:** Libraries _do not_ update on their own.  When a new version of GESI is released, you will have to manually update the `version` dropdown in the Libraries modal.

## FAQ

### How do I know if I have the latest version of GESI?

GESI will automatically update when a new version is released.  To see what changed visit the [forum thread](https://forums.eveonline.com/t/6-5-0-gesi-google-sheets-esi-library-now-with-some-post-endpoints/13406) or the [Github Releases](https://github.com/Blacksmoke16/GESI/releases) page.

**NOTE:  Changes in the ESI spec, such as adding/removing columns, name changes etc. may break your sheet.**

### How do I know what functions are available?

Check out the `function.ts` file under `src/script`.  This file lists all the functions available, as well as a description of  hat it returns and of each parameter.

### Why is my data not updating?

In order to improve performance, and reduce the number of requests to ESI (a user can only make 20,000 requests per day between all their sheets), GESI implements caching on the data returned from ESI.  The length of time that data will be cached depends on the function and when the data is expected to refresh on ESI's side.  

### How can I get more than 1 page of data at a time?

Use `-1` as the value for the page parameter.  This will return all pages, however it'll take longer to return.  If you see the error `Internal error when executing the custom function`, it means GESI reached the 30 second custom function execution time limit imposed by Google.  A solution for this would be to try calling the function again.  This will allow GESI to only fetch new data, and used the cached data for everything else.

### What do the function parameter types mean?

| Type    | Sample                             |
| ------- | ---------------------------------- |
| boolean | `true` or `false`                  |
| number  | `12`                               |
| string  | `"foo"` Notice the _double_ quotes |

Array types are denoted with a `[]` following the data type of the parameter.  An example of an array type could be `number[]` where a value for that would be `A1:A1` where this range is a column of numbers.

### Why does this cell contain all this random data?

As of now if an endpoint returns a property that is an array of objects nested inside the response, I am JSON stringifying it and displaying it in the column.  The `parseArray` allows you to parse that array of values and output it like an endpoint function does, wherever you want to.  You supply it with the name of the function the data is from, the column you are parsing, and the cell with the array data.

An example of this would be `parseArray("character_character_skills", "skills", B2)` where `B2` is the cell that contains the data.

### How do I know you're not stealing all my data?

For one, since everyone is using my Developer Application, it would be in violation of the [EVE Developer License Agreement](https://developers.eveonline.com/resource/license-agreement), specifically section `2.3.c/d`.

Secondly, the source for the add-on is still open source under `src/script/gesi.ts`.  The code is in Typescript, which gets compiled down to an earlier version of JavaScript by using `google/clasp` CLI tool.  Feel free to check it out for yourself.

Thirdly, the compiled code can be seen on [Google Scripts](https://script.google.com/a/mail.rmu.edu/d/1KjnRVVFr2KiHH55sqBfHcZ-yXweJ7iv89V99ubaLy4A7B_YH8rB5u0s3/edit?usp=sharing).

**HOWEVER,** I am collecting various debug information, on each request, for example:

```json
  {
    "id": 1,
    "datetime": "2018-08-22 00:21:03",
    "endpoint_name": "characters_character_assets",
    "character": "Blacksmoke17",
    "main_character": "Blacksmoke16",
    "sheet_id": "1XaFCiG8FdasLmnJ9jRfxG82slHA2UUaKeDwwLX-yo",
    "params": "{\"name\": \"\", \"page\": 1, \"opt_headers\": false}",
    "method": "GET",
    "path": "/v3/characters/2047918291/assets/?page=1",
    "data": null,
    "errors": null
  }
```

This is only used for debugging purposes, such as finding bugs, making pretty charts, etc.  Personal data, such as access_tokens, refresh_tokens, and response data are not logged.

### Using functions with multiple characters

A common use case is wanting to get the same data from multiple characters.  For example, getting the industry jobs of multiple characters into a nice, easy to manage format.  Currently this can be achieved by calling `=characters_character_industry_jobs()` once, then leaving some space and adding more for each additional characters with opt_headers disabled.  While it is an ok workaround it is not optimal, since there could be empty rows, not easily expandable/editable, etc.  A better alternative would be to define a new custom function `getJobs(character_names)` that will output the industry jobs of the given characters, in a single function call.

```JavaScript
/**
* Returns the combined industry jobs belonging to the given characters
* @param {string[]} character_names Character names to get jobs for.
* @param {boolean} opt_headers Default: True, Boolean if column headings should be listed or not.
* @return Combined industry job.
* @customfunction
*/
function getJobs(character_names, opt_headers) {
  var characters = character_names.split(",");
  var jobs = GESI.characters_character_industry_jobs(false, characters.shift(), opt_headers);
  characters.forEach(function(character) {
    jobs = jobs.concat(GESI.characters_character_industry_jobs(false, character.trim(), false));
  });
  return jobs;
}
```

Which would for three characters would be used like `=getJobs("Blacksmoke16, Character2, Character3")`.

This is of course just an example, but the general idea can be used as a template for other endpoint functions and uses.

## Contact Info

In-game:  Blacksmoke16  
Discord:  Blacksmoke16#0016
Discord Server: https://discordapp.com/invite/eEAH2et

## Copyright
 EVE Online and the EVE logo are the registered trademarks of CCP hf. All rights are reserved worldwide. All other 
 trademarks are the property of their respective owners. EVE Online, the EVE logo, EVE and all associated logos and designs are the intellectual property of CCP hf. All artwork, screenshots, characters, vehicles, storylines, world facts or other recognizable features of the intellectual property relating to these trademarks are likewise the intellectual property of CCP hf.    CCP hf. has granted permission to GESI to use EVE Online and all associated logos and designs for promotional and information purposes on its website but does not endorse, and is not in any way affiliated with, the GESI. CCP is in no way responsible for the content on or functioning of this website, nor can it be liable for any damage arising from the use of this website.
