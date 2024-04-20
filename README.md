# roslyn.nvim

This plugin adds support for the new Roslyn-based C# language server [introduced](https://devblogs.microsoft.com/visualstudio/announcing-csharp-dev-kit-for-visual-studio-code) in the [vscode C# extension](https://github.com/dotnet/vscode-csharp).

## Dependencies

Ideally I would like to depend on the Dotnet SDK and everything else to be optional. But for now:

- Dotnet SDK (Tested with .net7).
- [`nvim-lspconfig`](https://github.com/neovim/nvim-lspconfig) for some path utility functions.
- Neovim nightly required. Tested on `831d662ac6756cab4fed6a9b394e68933b5fe325` but anything after August 2023 would probably work.
- `markdown_inline` tree-sitter parser for good hover support.

## Setup

To install this fork, just install `krsma33/roslyn.nvim` using your plugin manager. 
To install from original creator, just install `jmederosalvarado/roslyn.nvim` using your plugin manager. 

```lua
require("roslyn").setup({
    dotnet_cmd = "dotnet", -- this is the default
    roslyn_version = "4.8.0-3.23475.7", -- this is the default
    on_attach = <on_attach you would pass to nvim-lspconfig>, -- required
    capabilities = <capabilities you would pass to nvim-lspconfig>, -- required
})
```

### Settings

Settings can be passed to the setup function. The following settings are available [vscode-csharp unit tests link](https://github.com/dotnet/vscode-csharp/blob/main/test/unitTests/configurationMiddleware.test.ts):

```lua
code_style.formatting.new_line.insert_final_newline

csharp|background_analysis.dotnet_analyzer_diagnostics_scope
csharp|background_analysis.dotnet_compiler_diagnostics_scope
csharp|code_lens.dotnet_enable_references_code_lens
csharp|code_lens.dotnet_enable_tests_code_lens
csharp|code_style.formatting.indentation_and_spacing.indent_size
csharp|code_style.formatting.indentation_and_spacing.indent_style
csharp|code_style.formatting.indentation_and_spacing.tab_width
csharp|code_style.formatting.new_line.end_of_line
csharp|completion.dotnet_provide_regex_completions
csharp|completion.dotnet_show_completion_items_from_unimported_namespaces
csharp|completion.dotnet_show_name_completion_suggestions
csharp|highlighting.dotnet_highlight_related_json_components
csharp|highlighting.dotnet_highlight_related_regex_components
csharp|implement_type.dotnet_insertion_behavior
csharp|implement_type.dotnet_property_generation_behavior
csharp|inlay_hints.csharp_enable_inlay_hints_for_implicit_object_creation
csharp|inlay_hints.csharp_enable_inlay_hints_for_implicit_variable_types
csharp|inlay_hints.csharp_enable_inlay_hints_for_lambda_parameter_types
csharp|inlay_hints.csharp_enable_inlay_hints_for_types
csharp|inlay_hints.dotnet_enable_inlay_hints_for_indexer_parameters
csharp|inlay_hints.dotnet_enable_inlay_hints_for_literal_parameters
csharp|inlay_hints.dotnet_enable_inlay_hints_for_object_creation_parameters
csharp|inlay_hints.dotnet_enable_inlay_hints_for_other_parameters
csharp|inlay_hints.dotnet_enable_inlay_hints_for_parameters
csharp|inlay_hints.dotnet_suppress_inlay_hints_for_parameters_that_differ_only_by_suffix
csharp|inlay_hints.dotnet_suppress_inlay_hints_for_parameters_that_match_argument_name
csharp|inlay_hints.dotnet_suppress_inlay_hints_for_parameters_that_match_method_intent
csharp|quick_info.dotnet_show_remarks_in_quick_info
csharp|symbol_search.dotnet_search_reference_assemblies

mystery_language|Highlighting.dotnet_highlight_related_json_components
mystery_language|background_analysis.dotnet_analyzer_diagnostics_scope
mystery_language|background_analysis.dotnet_compiler_diagnostics_scope
mystery_language|code_lens.dotnet_enable_references_code_lens
mystery_language|code_lens.dotnet_enable_tests_code_lens
mystery_language|completion.dotnet_provide_regex_completions
mystery_language|completion.dotnet_show_completion_items_from_unimported_namespaces
mystery_language|completion.dotnet_show_name_completion_suggestions
mystery_language|highlighting.dotnet_highlight_related_regex_components
mystery_language|implement_type.dotnet_insertion_behavior
mystery_language|implement_type.dotnet_property_generation_behavior
mystery_language|quick_info.dotnet_show_remarks_in_quick_info
mystery_language|symbol_search.dotnet_search_reference_assemblies

navigation.dotnet_navigate_to_decompiled_sources

text_editor.tab_width
````

Example enabling inlay hints:

```lua
require("roslyn").setup({
    dotnet_cmd = "dotnet", -- this is the default
    roslyn_version = "4.8.0-3.23475.7", -- this is the default
    on_attach = <on_attach you would pass to nvim-lspconfig>, -- required
    capabilities = <capabilities you would pass to nvim-lspconfig>, -- required
    settings = {
      ["csharp|inlay_hints"] = {
        csharp_enable_inlay_hints_for_implicit_object_creation = true,
        csharp_enable_inlay_hints_for_implicit_variable_types = true,
        csharp_enable_inlay_hints_for_lambda_parameter_types = true,
        csharp_enable_inlay_hints_for_types = true,
        dotnet_enable_inlay_hints_for_indexer_parameters = true,
        dotnet_enable_inlay_hints_for_literal_parameters = true,
        dotnet_enable_inlay_hints_for_object_creation_parameters = true,
        dotnet_enable_inlay_hints_for_other_parameters = true,
        dotnet_enable_inlay_hints_for_parameters = true,
        dotnet_suppress_inlay_hints_for_parameters_that_differ_only_by_suffix = true,
        dotnet_suppress_inlay_hints_for_parameters_that_match_argument_name = true,
        dotnet_suppress_inlay_hints_for_parameters_that_match_method_intent = true,
      },
    },
})
```

## Usage

Before trying to use the language server you should install it. You can do so by running `:CSInstallRoslyn` which will install the configured version of the plugin in neovim's datadir.

1. Upon opening a C# file, the plugin will look for a `.sln` file in parent directories until it finds one. Make sure to have a `.sln` file somewhere in a parent dir, there is no support yet for `.csproj` only projects. 
2. If it only finds one `.sln` file, it will use that to start the server. If it finds multiple, it will ask you to choose one before starting the server. When multiple `.sln` files are found for a file, you can use the command `:CSTarget` to change the target for the buffer at any point.
3. You'll see two notifications if everything goes well. The first one will say `Roslyn client initialized for target <target>`, which means the server is running, but it will just start indexing your `sln`. The second one will say `Roslyn project initialization complete`, it means that the server indexed your `sln`, only after you see the second notification will the `go to definition` and other lsp features be available.

## Features

Please note that some features from the vscode extension might not yet be supported by this plugin. Most of them are part of the roadmap, however I don't use vscode myself, so I'm not aware of all the features available, feel free to open an issue if you notice something is missing.
