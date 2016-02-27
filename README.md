# Marker

Marker is a markup language parser designed for two needs:

1.  Need to mimic MediaWiki syntax
2.  Need to provide multiple output formats

Mimicing MediaWiki syntax is not exact. One reason is that the MediaWiki parser itself is very complicated and handles many cases specially. It would be very difficult to exactly copy the MediaWiki parser and it probably wouldn't be worth the time because MediaWiki is intended for a wiki and needs to be adapted to be used as a markup lanaguage--especially for multiple output formats. The purpose of mimicing MediaWiki syntax is so that users don't have to learn more than one markup language, so the implementation doesn't _need_ to be exact anyway.

Marker differs from MediaWiki in several ways, because it is a grammar-based implementation. The grammar is written as a [Treetop](http://treetop.rubyforge.org/) parsing expression grammar ([PEG](http://en.wikipedia.org/wiki/Parsing_expression_grammar)).

Not implemented:

1.  Table of contents
2.  Tables

## Use

Parsing is done with either Marker.parse or Marker.parse_file. Both parse methods will return a parse tree that has <tt>to_html</tt> and <tt>to_s</tt> methods that "render" the markup. Both render methods will accept an options hash.

Example:

<pre>>> require 'marker'
=> true
>> m = Marker.parse "== heading ==\nparagraph with '''bold''' text"
=> Markup+...
>> puts m.to_s
heading
--------------------------------------------------------------------------------
paragraph with *bold* text
=> nil
>> puts m.to_html
<h2>heading</h2>
<p>paragraph with <b>bold</b> text</p>
=> nil
</pre>

### Templates

Templates are implemented as method calls to a templates module. Each method in the templates module is considered a template and can be called using the "<tt>{{template_name}}</tt>" syntax. Each template method is expected to take three arguments: the render format (<tt>:html</tt> or <tt>:text</tt>), an array of positional parameters, and a hash of named parameters. For example,

<pre>module MyTemplates
  def logo( format, pos_params, name_params )
    case format
    when :html
      '<img src="/images/logo.png" />'
    else
      ''
    end
  end
end
</pre>

Template modules are passed to Marker by setting the <tt>templates</tt> property:

<pre>require 'my_templates'
require 'marker'

Marker.templates = Templates
</pre>

If no template method is found, the template call is printed for debugging:

<pre>>> puts Marker.parse( '{{t|one|two|name=val}}' ).to_s
render:t( :text, ["one", "two"], {"name"=>"val"} )
</pre>

Template names from markup are converted to lower case and have spaces replaced with underscores to match ruby method naming conventions and to be case insensitive for markup writers. For example,

<pre>"{{ My Template }}"  => :my_template
"{{NaMe}}"           => :name
</pre>

### Internal Links

Internal links are implemented as links with default prefixes. The link prefix is specified by setting the <tt>link_base</tt> property:

<pre>require 'marker'

Marker.link_base = 'http://example.com/pages/'

>> puts Marker.parse( '[[target|name]]' ).to_html
<p><a href='http://example.com/pages/target'>name</a></p>
</pre>

The link target is appended to the link prefix, along with a beginning '/'. If no link base is given, links are just the link target with a beginning '/'. The link base can also be given as a render option.

### Unlabelled Links

Unlabelled, or "bare" links are detected if they start with a recognized URL scheme such as <tt>http</tt> or <tt>https</tt>. The URL is used as the link text.

## Command Line Program

Marker comes with a command-line program that will render both HTML and text. If no input file is given, it reads from stdin.

<pre>Usage: marker [--format (html|text)] [input-file]
</pre>

## License

Marker is copyright 2009 Ryan Blue and distributed under the terms of the GNU General Public License (GPL). See the LICENSE file for further information on the GPL, or visit [http://creativecommons.org/licenses/GPL/2.0/](http://creativecommons.org/licenses/GPL/2.0/).
