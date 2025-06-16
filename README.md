# JUCE Tutorials

These are the markdown sources for the [JUCE framework](https://github.com/juce-framework/JUCE) tutorials published at https://juce.com/learn/tutorials/.

## Project Structure

- Only tutorials in the `markdown` folder will be published.
- All images live in the `_images` dir.
- Image urls should look like `./some_image.png`.
- Any folder/file starting with \_ (underscore) or . (dot) won’t be published.

## Markdown specifics

The markdown implementation is "GitHub Flavored" [Parsedown](https://parsedown.org/) with [Markdown Extra](https://michelf.ca/projects/php-markdown/extra/).

### Callouts

As with GFM, the callout syntax is:

```
> [!TIP]
> The code presented here is broadly similar to the **PlugInSamples** from the JUCE Examples.
```

Which will look like:

> [!TIP]
> The code presented here is broadly similar to the **PlugInSamples** from the JUCE Examples.

The options are `TIP`, `WARNING`, `NOTE`. Tutorial exercises are treated as `NOTE`.

### Multiple callouts in a row

If you want multiple callouts in a row, there's currently a bug which will bundle them together. To separate them, you can put an html comment between them, like so:

```
> [!TIP]
> First tip

<!-- -->

> [!TIP]
> Second tip
```
