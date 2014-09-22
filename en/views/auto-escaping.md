# Auto-escaping

You might have noticed on other pages in the manual that li3 uses the short tag syntax to output the contents of a view variable. This syntax is a bit misleading, as li3 does not actually depend on or use short tags: this output behavior works a bit differently from how it seems.  The modified functionality is designed to save time and improve your application's security by escaping output.  Escaping output is a core strategy of defense in depth.

When the view layer is rendered, each template is processed by a tokenizer before it is compiled into its final form. During this step something like this with "short tags":

```
<?=$variable; ?>
```

Is translated into something like this, which is properly escaped:

```
<?php echo $h($variable); ?>
```

The `$h()` function is there to escape HTML output. This mechanism provides an easy and effective way to make sure all dynamically-generated data is displayed safely in your HTML template.

We highly recommend using the `<?= ...; ?>` syntax in your views, as it aids greatly in hardening your application against cross-site scripting (XSS) and related attack techniques.

>#####Note:#####

>One exception to this rule is when a line of template code references the `$this` object. In those cases, output is written directly to the template, rather than being filtered through `$h()`. This is so that content from helpers is not double-escaped. As such, the following two statements are equivalent:

```
<?=$this->form->create(); ?>

<?php echo $this->form->create(); ?>
```

This is an important consideration when accessing properties and methods from the template renderer. If you intend to echo content directly from `$this` which is not coming from a helper (this is not a common occurence), you must manually escape it, like so:

```
<?php echo $h($this->foo); ?>
```