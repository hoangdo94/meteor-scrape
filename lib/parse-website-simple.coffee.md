# ParseWebsiteSimple
This core module takes an HTML string and returns a proper data object. The
following NPM modules are used:

    cheerio = Npm.require "cheerio"
    readability = Npm.require "readabilitySAX"

The API of this module includes a central `run()` method.

    @ParseWebsiteSimple = share.ParseWebsiteSimple = (html) ->
      txt = extractFromText html
      dom = extractFromDOM html
      mergeResults txt, dom

## Extract the raw data from the plaintext and analyze it

    extractFromText = (html) ->
      data = {}
      data.text = readability.process(html, {type: "text"}).text
      data.text = "" unless data.text?.length > 50
      return data

## DOM Parsing

Every bit of valuable data is needed, so let's get dirty and fish for more
stuff by hand. To make scraping a bit more sane, the package
[cheerio](https://www.npmjs.com/package/cheerio) is used. It's kind of
jQuery, but on the server. This way, CSS3 selectors can be leveraged instead
of nasty XPaths or unreadable RegExps.

    extractFromDOM = (html) ->
      $ = cheerio.load html
      data = {}
      data.text = findText $
      data.metaKeywords = findTags $
      return data

    findTags = ($) ->
      str = $("meta[name='keywords']").attr("content")
      tags = []
      if str
        lang = Text.detectLanguage str
        tags = if /;/.test str then str.split ';' else str.split ','
      return tags

    findText = ($) ->
      $("h1,h2,h3,h4,h5,h6,p,article").map((i,e) -> $(e).text()).get().join("\n\n")


## Merge the results

Join the results from the TextParser and DomParser into one uniform
result object. Pick the best results if there is some overlap.

    mergeResults = (txt, dom) ->
      data = {}
      data.text = Text.clean(txt.text or dom.text)
      data.metaKeywords = dom.metaKeywords
      return data