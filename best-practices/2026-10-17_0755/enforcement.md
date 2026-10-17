# Enforcement

> **INTERIM snapshot (2026-10-17 22:50, Asia/Manila).** Claude collection chunks and the stricter adoption re-run are still pending. Levels and counts will change when they are folded in. See README.md.

Which tool rule enforces each listed practice. Rules without a tool are enforced manually in review.

595 of 5803 listed rules have a tool rule.

| Level | Rule | Enforced by |
|---|---|---|
| Must | Read environment variables in configuration files and read those settings through config() in application code | larastan:NoEnvCallsOutsideOfConfigRule, shift:workbench/convert-env-to-config |
| Must | Use framework fakes at framework boundaries | partial: rector:ReplaceExpectsMethodsInTestsRector, partial: rector:ReplaceWithoutJobsEventsAndNotificationsWithFacadeFakeRector |
| Must | Configure middleware, exception handling and route loading in bootstrap/app.php in streamlined applications | partial: shift:slimmer/bootstrap |
| Must | Scope dispatch uniqueness to the logical operation | partial: larastan:UniqueJobDeclaresUniqueIdRule |
| Must | Return anonymous migration classes | rector:AnonymousMigrationsRector, shift:workbench/adopt-anonymous-migrations |
| Must | Serialize model identifiers rather than live dependencies | partial: larastan:JobWithModelPropertyDeclaresSerializesModelsRule |
| Must | Use application environment predicates for environment checks | rector:AppEnvironmentComparisonToParameterRector |
| Must | Do not store request-specific state in Octane singletons or static properties | partial: larastan:OctaneCompatibilityRule |
| Must | Include CSRF tokens in state-changing forms | partial: shift:workbench/blade/csrf |
| Must | Follow PHPUnit discovery and naming conventions | pint:php_unit_method_casing, shift:workbench/format-test-cases |
| Must | Use the Laravel Vite asset pipeline | partial: shift:vite-converter |
| Must | Declare custom Eloquent collections with #[CollectedBy] for new Laravel 13 code | rector:CollectedByPropertyToCollectedByAttributeRector |
| Must | Do not rely on batch or bulk dispatch uniqueness | partial: larastan:NoBatchedUniqueJobRule |
| Must | Use supported LIKE helpers with deliberate case semantics | partial: rector:WhereToWhereLikeRector |
| Must | Load relationship counts with withCount() instead of hydrating children | partial: larastan:NoUnnecessaryCollectionCallRule |
| Must | Resolve current request/container/config dependencies at use time rather than retaining stale instances in Octane singletons | partial: larastan:OctaneCompatibilityRule |
| Must | Declare nullable parameters explicitly | pint:nullable_type_declaration_for_default_null_value |
| Must | Compare shared secrets with timing-safe equality | partial: rector:StaticCallToFuncCallRector |
| Must | Use class-based factories and named state methods | rector:FactoryApplyingStatesRector, rector:FactoryDefinitionRector, rector:FactoryFuncCallToStaticCallRector |
| Must | Use $request->input() instead of $request->get(); input() supports dot notation for nested values | rector:RenameMethodRector, shift:public/laravel13/request-input |
| Must | Stay current within supported releases | partial: shift:public/linter/current-version |
| Must | Select constrained eager-load columns inside the callback | partial: larastan:RelationExistenceRule |
| Must | List only defined computed accessors in model appends | larastan:ModelAppendsRule |
| Must | Use valid delimited regular expressions | partial: larastan:PHPStan\Rules\Regexp\RegularExpressionPatternRule |
| Must | Keep classes compatible with PSR-4 autoloading | partial: pint:psr_autoloading |
| Must | Declare PHPUnit metadata with attributes | shift:phpunit/attributes |
| Must | Replace legacy Laravel global helpers with Arr and Str | rector:FuncCallToStaticCallRector |
| Must | Return every provided service binding from provides() in a deferrable service provider | partial: larastan:DeferrableServiceProviderMissingProvidesRule |
| Must | Declare Attribute accessors and attributed local scopes protected | larastan:NoPublicModelScopeAndAccessorRule, rector:MakeModelAttributesAndScopesProtectedRector |
| Must | Replace the removed model dates property with date or datetime casts | rector:UnifyModelDatesWithCastsRector, shift:workbench/convert-dates-property |
| Must | Use Laravel request, server and session APIs instead of PHP request/session superglobals | rector:RequestVariablesToRequestFacadeRector, rector:ServerVariableToRequestFacadeRector, rector:SessionVariableToSessionFacadeRector |
| Must | Use HasMiddleware::middleware() for controller-owned middleware in new streamlined applications | shift:upgrade-12/controller-middleware |
| Must | Use explicit nullable parameter types | pint:nullable_type_declaration, pint:nullable_type_declaration_for_default_null_value |
| Must | Honor batch cancellation before doing work | partial: larastan:BatchableJobChecksCancellationRule |
| Must | Add Batchable to jobs submitted to batches | larastan:BatchedJobIsBatchableRule |
| Must | Enqueue transaction-dependent jobs after commit | partial: larastan:JobDispatchedInTransactionUsesAfterCommitRule |
| Must | Replace obsolete framework event-name strings with event classes | rector:EventStringToClassConstantRector |
| Must | Use explicit HTML assertions for unescaped response assertions | rector:AssertSeeToAssertSeeHtmlRector |
| Must | Remove debug calls before committing | partial: rector:RemoveDumpDataDeadCodeRector |
| Must | Use blank and filled with intentional value semantics | partial: rector:EmptyToBlankAndFilledFuncRector |
| Should | Define model casts in casts(), preserving an established project convention | rector:ModelCastsPropertyToCastsMethodRector |
| Should | Inject container-managed collaborators explicitly where ordinary dependency injection is available | partial: rector:ArgumentFuncCallToMethodCallRector |
| Should | Use subject-specific assertions | partial: shift:phpunit-13/type-specific-assertions |
| Should | Parameterize equivalent test cases | partial: shift:pest-converter/datasets |
| Should | Inject dependencies at replaceable boundaries | partial: rector:StaticCallToMethodCallRector |
| Should | Use framework contracts for replaceable service dependencies | partial: rector:ArgumentFuncCallToMethodCallRector |
| Should | Use Laravel 13 model attributes when adopting the attribute convention | partial: rector:EmptyGuardedPropertyToUnguardedAttributeRector, partial: rector:FillablePropertyToFillableAttributeRector, partial: rector:GuardedPropertyToGuardedAttributeRector, partial: shift:workbench/laravel-property-attributes |
| Should | Declare native parameter and return contracts | partial: larastan:PHPStan\Rules\Methods\MissingMethodParameterTypehintRule, partial: larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule, partial: larastan:PHPStan\Rules\Functions\MissingFunctionParameterTypehintRule, partial: larastan:PHPStan\Rules\Functions\MissingFunctionReturnTypehintRule |
| Should | Write multi-rule validation as arrays | partial: rector:ValidationRuleArrayStringValueToArrayRector, partial: shift:workbench/adopt-validation-arrays |
| Should | Extract complex or reusable validation into Form Requests | partial: shift:workbench/convert-controller-validation |
| Should | Reuse suitable Laravel and application capabilities | partial: shift:laravel-linter/leverage-features |
| Should | Use Attribute-returning methods for new model accessors and mutators unless a legacy style is established | rector:MigrateToSimplifiedAttributeRector, shift:workbench/adopt-attribute-accessors-mutators |
| Should | Keep ordinary application models in App\Models unless the project uses an established modular layout | shift:public/namespace-models/move |
| Should | Use automatic command discovery in standard directories | shift:workbench/leverage-command-autoloading |
| Should | Inject persistent collaborators through constructors | partial: rector:ArgumentFuncCallToMethodCallRector, partial: rector:StaticCallToMethodCallRector |
| Should | Use readable PHPUnit test names | pint:php_unit_method_casing |
| Should | Run application checks in CI | partial: shift:public/ci/tests, partial: shift:public/ci/style |
| Should | Publish only configuration files and options that the application customizes, preserving framework merge behavior | partial: shift:public/slimmer/configuration |
| Should | Import class and facade symbols explicitly instead of relying on global aliases or repeated fully qualified references | partial: rector:RenameClassRector, partial: pint:fully_qualified_strict_types, partial: shift:workbench/replace-global-facades |
| Should | Prefer back() for straightforward redirects to the previous page | rector:RedirectBackToBackHelperRector |
| Should | Space the logical NOT operator consistently | pint:not_operator_with_successor_space |
| Should | Use expectations for values in Pest | partial: shift:pest-converter/expectations |
| Should | Prefer named status assertions | rector:AssertStatusToAssertMethodRector |
| Should | Check type coverage | partial: larastan:PHPStan\Rules\Functions\MissingFunctionParameterTypehintRule, partial: larastan:PHPStan\Rules\Functions\MissingFunctionReturnTypehintRule, partial: larastan:PHPStan\Rules\Methods\MissingMethodParameterTypehintRule, partial: larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule, partial: larastan:PHPStan\Rules\Properties\MissingPropertyTypehintRule |
| Should | Annotate concrete model factory types | rector:AddExtendsAnnotationToModelFactoriesRector |
| Should | Use equivalent Laravel 13 model attributes when adopting the declarative configuration style | partial: rector:AppendsPropertyToAppendsAttributeRector, partial: rector:ConnectionPropertyToConnectionAttributeRector, partial: rector:DateFormatPropertyToDateFormatAttributeRector, partial: rector:HiddenPropertyToHiddenAttributeRector, partial: rector:RouteKeyMethodToRouteKeyAttributeRector, partial: rector:TablePropertyToTableAttributeRector, partial: shift:workbench/laravel-property-attributes |
| Should | Use Url attributes for shareable Livewire state | rector:LivewireComponentQueryStringToUrlAttributeRector |
| Should | Reference controller actions with tuple notation [Controller::class, 'method'] instead of 'Controller@method' strings | rector:RouteActionCallableRector, shift:workbench/adopt-class-based-routes |
| Should | Promote ordinary constructor-only properties | partial: rector:ClassPropertyAssignToConstructorPromotionRector, partial: shift:workbench/adopt-new-in-initializers |
| Should | Skip asset manifest lookup in server tests | partial: shift:public/vite/test-helpers |
| Should | Use a Form Request for each meaningful input action | partial: shift:workbench/convert-controller-validation |
| Should | Use Laravel current-date helpers | partial: rector:NowFuncWithStartOfDayMethodCallToTodayFuncRector |
| Should | Keep magic property annotations valid | larastan:PHPStan\Rules\Classes\PropertyTagRule, larastan:PHPStan\Rules\Classes\PropertyTagTraitRule, larastan:PHPStan\Rules\Classes\PropertyTagTraitUseRule |
| Should | Commit the selected formatter configuration and enforce it consistently in CI | partial: shift:public/ci/style |
| Should | Document custom facade methods with accurate @method annotations | partial: larastan:PHPStan\Rules\Classes\MethodTagRule |
| Should | Use Artisan console components under the styled-output convention | partial: rector:UseComponentPropertyWithinCommandsRector |
| Should | Use dedicated Blade authentication directives | shift:workbench/modernize-blade-directives |
| Should | Use DeferrableProvider for deferred service providers | rector:PropertyDeferToDeferrableProviderToRector |
| Should | Declare the Laravel majors a package actually supports | partial: shift:workbench/bump-laravel-package-dependencies |
| Should | Keep streamlined application base controllers minimal | partial: shift:upgrade-12/base-controller-traits |
| Should | Write inheritdoc as an inline PHPDoc tag | pint:phpdoc_tag_type |
| Should | Build response metadata fluently | shift:workbench/adopt-fluent-responses |
| Should | Use fluent strings for sequential transformations | shift:workbench/adopt-stringables |
| Should | Enable strict scalar argument checking | pint:declare_strict_types, shift:workbench/declare-strict-types |
| Should | Use the adopted Laravel PHPDoc alignment | pint:phpdoc_align |
| Should | Keep PHPStan ignore annotations valid | partial: larastan:PHPStan\Rules\Ignore\IgnoreParseErrorRule |
| Should | Represent stored callbacks with Closure or callable PHPDoc | partial: larastan:PHPStan\Rules\Properties\InvalidCallablePropertyTypeRule |
| Should | Reference class names with class constants | shift:workbench/use-class-constant |
| Should | Use lookup defaults only when semantics match | partial: rector:ApplyDefaultInsteadOfNullCoalesceRector |
| Should | Preserve command and supported framework metadata when adopting Laravel 13 PHP attributes | partial: rector:AliasesPropertyToAliasesAttributeRector, partial: rector:DescriptionPropertyToDescriptionAttributeRector, partial: rector:HelpPropertyToHelpAttributeRector, partial: rector:SignaturePropertyToSignatureAttributeRector |
| Should | Prefer bound class or contract identifiers over equivalent legacy string container keys | partial: rector:ReplaceServiceContainerCallArgRector |
| Should | Use return-type-hinted binding closures to avoid repeating the abstract when adopting inferred binding syntax | rector:ContainerBindConcreteWithClosureOnlyRector |
| Should | Use protected #[Scope] methods for new local scopes unless the project retains legacy scope naming | rector:ScopeNamedClassMethodToScopeAttributedClassMethodRector, shift:workbench/laravel-local-scope-attribute |
| Should | Declare a precise array return contract for model casts() | partial: larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule |
| Should | Use computed properties for derived page data | partial: rector:LivewireComponentComputedMethodToComputedAttributeRector |
| Should | Use braces for every control-structure body | pint:control_structure_braces |
| Should | Use intention-revealing prefix and suffix checks | partial: rector:SubStrToStartsWithOrEndsWithStaticMethodCallRector |
| Should | Complete the selected Tailwind build integration | partial: shift:public/vite/postcss |
| Should | Write string concatenations without spaces around the dot operator | pint:concat_space |
| Should | Check formatting in CI | partial: shift:public/ci/style |
| Should | Bound uniqueness lock lifetime | partial: larastan:UniqueJobDeclaresUniqueForRule |
| Should | Use Laravel 13 queue attributes consistently when adopting them | partial: rector:BackoffPropertyToBackoffAttributeRector, partial: rector:DelayPropertyToDelayAttributeRector, partial: rector:DeleteWhenMissingModelsPropertyToDeleteWhenMissingModelsAttributeRector, partial: rector:FailOnTimeoutPropertyToFailOnTimeoutAttributeRector, partial: rector:JobConnectionPropertyToJobConnectionAttributeRector, partial: rector:MaxExceptionsPropertyToMaxExceptionsAttributeRector, partial: rector:QueuePropertyToQueueAttributeRector, partial: rector:TimeoutPropertyToTimeoutAttributeRector, partial: rector:TriesPropertyToTriesAttributeRector, partial: rector:UniqueForPropertyToUniqueForAttributeRector |
| Should | Declare valid PHPDoc generic contracts | partial: larastan:PHPStan\Rules\Generics\ClassTemplateTypeRule, partial: larastan:PHPStan\Rules\Generics\FunctionTemplateTypeRule, partial: larastan:PHPStan\Rules\Generics\InterfaceTemplateTypeRule, partial: larastan:PHPStan\Rules\Generics\MethodTagTemplateTypeRule, partial: larastan:PHPStan\Rules\Generics\MethodTagTemplateTypeTraitRule, partial: larastan:PHPStan\Rules\Generics\MethodTemplateTypeRule |
| Should | Remove unused imports | pint:no_unused_imports |
| Should | Use canonical collection method names | rector:UnaliasCollectionMethodsRector |
| Should | Remove identity-only with helper calls | larastan:NoUselessWithFunctionCallsRule, rector:RemoveRedundantWithCallsRector |
| Should | Retain failed browser diagnostics | partial: shift:ci-generator/dusk-job |
| Should | Prefer the shared fake helper in modern factories | rector:ReplaceFakerInstanceWithHelperRector |
| Should | Call Faker formatters as methods | rector:ReplaceFakerPropertyFetchWithMethodCallRector |
| Should | Use fluent rule builders for complex parameterized constraints | partial: shift:workbench/convert-validation-rules |
| Should | Migrate Form Request behavior properties to Laravel 13 attributes | partial: rector:ErrorBagPropertyToErrorBagAttributeRector, partial: rector:StopOnFirstFailurePropertyToStopOnFirstFailureAttributeRector |
| Should | Remove unused dependencies after checking usage | partial: shift:workbench/remove-unused-composer-dependencies |
| Should | Match generic interface annotations to real contracts | larastan:PHPStan\Rules\Generics\ClassAncestorsRule, larastan:PHPStan\Rules\Generics\EnumAncestorsRule, larastan:PHPStan\Rules\Generics\InterfaceAncestorsRule |
| Should | Use early returns for invalid or empty input | partial: rector:ChangeNestedIfsToEarlyReturnRector |
| Should | Avoid PHP 8.5 global helper name collisions | partial: rector:FuncCallToStaticCallRector |
| Should | Match CI PHP versions, extensions, and container services to the project’s supported development and runtime requirements | partial: shift:public/ci/php-environment |
| Should | Configure application exception policy through withExceptions() in the streamlined bootstrap | partial: shift:slimmer/bootstrap |
| Should | Start composable conditional model queries with query() and follow the established chain style | partial: larastan:NoImplicitQueryBuilderCallRule, partial: rector:EloquentMagicMethodToQueryBuilderRector |
| Should | Push large-data filtering into SQL rather than filtering hydrated models in PHP | partial: larastan:NoUnnecessaryCollectionCallRule |
| Should | Use request->user() when a request object is already available | larastan:NoAuthFacadeInRequestScopeRule, larastan:NoAuthHelperInRequestScopeRule |
| Should | Prefer composition for reusable model/controller behavior over deep custom base-class hierarchies | partial: shift:public/linter/composition |
| Should | Use PHP nullsafe access for simple optional chains | rector:OptionalToNullsafeOperatorRector, shift:workbench/convert-optional-to-nullsafe |
| Should | Give controller actions concrete Inertia or redirect response return types | partial: larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule |
| Should | Use first-class callables for simple method references | partial: rector:ArrayToFirstClassCallableRector |
| Should | Use strict value comparisons | partial: pint:strict_comparison, partial: pint:strict_param |
| Should | Choose collection extraction semantics deliberately | partial: larastan:NoUnnecessaryEnumerableToArrayCallsRule, partial: rector:ConvertEnumerableToArrayToAllRector |
| Should | Automate import ordering and unused-import cleanup | pint:ordered_imports, pint:no_unused_imports |
| Should | Replace legacy job retry configuration names | rector:RenameMethodRector, rector:RenamePropertyRector |
| Should | Use supported braced string interpolation | pint:simple_to_complex_string_variable |
| Should | Follow established test conventions | partial: pint:php_unit_method_casing |
| Should | Share Faker generators across fixture creation | partial: rector:ReplaceFakerInstanceWithHelperRector |
| Should | Use an explicit consistent request-access convention | partial: shift:workbench/normalize-request-access |
| Should | Remove no-op value helper calls | larastan:NoUselessValueFunctionCallsRule, rector:RemoveRedundantValueCallsRector |
| Should | Separate class members consistently | pint:class_attributes_separation |
| Should | Remove obsolete package PHPDoc tags | pint:phpdoc_no_package |
| Should | Use inheritdoc only for inherited contracts | pint:phpdoc_no_useless_inheritdoc |
| Should | Avoid duplicate trait members | larastan:PHPStan\Rules\Classes\DuplicateTraitDeclarationRule |
| Should | Import resolvable traits | larastan:PHPStan\Rules\Classes\ExistingClassInTraitUseRule |
| Should | Reference existing interfaces in inheritance declarations | larastan:PHPStan\Rules\Classes\ExistingClassesInClassImplementsRule, larastan:PHPStan\Rules\Classes\ExistingClassesInInterfaceExtendsRule |
| Should | Keep magic method annotations valid | larastan:PHPStan\Rules\Classes\MethodTagRule, larastan:PHPStan\Rules\Classes\MethodTagTraitRule, larastan:PHPStan\Rules\Classes\MethodTagTraitUseRule |
| Should | Use classes for PHP attribute types | larastan:PHPStan\Rules\Classes\TraitAttributeClassRule |
| Should | Remove trait conditions proven constant in consumers | larastan:PHPStan\Rules\Comparison\ConstantConditionInTraitRule |
| Should | Catch valid throwable types | larastan:PHPStan\Rules\Exceptions\CaughtExceptionExistenceRule |
| Should | Match generic trait annotations to imported traits | larastan:PHPStan\Rules\Generics\UsedTraitsRule |
| Should | Declare interface methods public | larastan:PHPStan\Rules\Methods\MethodVisibilityInInterfaceRule |
| Should | Implement every required method in concrete types | larastan:PHPStan\Rules\Methods\MissingMethodImplementationRule, larastan:PHPStan\Rules\Methods\OverridingMethodRule |
| Should | Preserve inherited exception contracts | larastan:PHPStan\Rules\Exceptions\MethodThrowTypeCovarianceRule |
| Should | Replace obsolete Laravel type names | partial: rector:RenameClassRector |
| Should | Depend on the cache repository contract for store operations | rector:RenameClassRector |
| Should | Use explicit methods on contract-typed cache repositories | rector:ArrayDimFetchToMethodCallRector |
| Should | Provide private Composer credentials through deployment or CI secrets before dependency installation | partial: shift:public/ci/private-packages |
| Should | Use Arr::get() for nested optional array reads while preserving the original missing-value semantics | rector:ArrayToArrGetRector |
| Should | Read environment-backed settings through env() inside configuration files instead of direct $_ENV access | rector:EnvVariableToEnvHelperRector |
| Should | Use self for enum and final-class member references when late static binding cannot change the target | pint:self_static_accessor |
| Should | Use array-unpacking key types supported by the configured PHP runtime | larastan:PHPStan\Rules\Arrays\ArrayUnpackingRule |
| Should | Use the App facade for container interaction outside providers when adopting the facade idiom | partial: rector:HelperFuncCallToFacadeClassRector |
| Should | Import the Schema facade explicitly | rector:RenameClassRector |
| Should | Rename frontend environment variables during Vite migration | shift:public/vite/environment |
| Should | Use explicit NULL predicates | rector:WhereNullComparisonToWhereNullRector |
| Should | Follow Eloquent schema conventions for new models unless domain or legacy constraints require overrides | partial: shift:workbench/remove-redundant-table-name |
| Should | Instantiate unsaved models with new when no factory behavior is needed | larastan:NoModelMakeRule |
| Should | Call parent::boot() when overriding model boot() | rector:AddParentBootToModelClassMethodRector |
| Should | Use lowercase canonical PHP casts with one space before the operand | pint:cast_spaces, pint:lowercase_cast, pint:no_short_bool_cast, pint:short_scalar_cast |
| Should | Write the PHP scope-resolution operator without surrounding spaces | pint:no_space_around_double_colon |
| Should | Replace the removed unset cast with explicit null or unset as appropriate | larastan:PHPStan\Rules\Cast\UnsetCastRule, pint:no_unset_cast |
| Should | Pass only string-convertible values to output and native string-conversion operations | larastan:PHPStan\Rules\Cast\EchoRule, larastan:PHPStan\Rules\Cast\InvalidPartOfEncapsedStringRule, larastan:PHPStan\Rules\Cast\PrintRule, larastan:PHPStan\Rules\Functions\ImplodeParameterCastableToStringRule, larastan:PHPStan\Rules\Functions\ParameterCastableToStringRule |
| Should | Use conversions supported by the source and required target types | larastan:PHPStan\Rules\Cast\InvalidCastRule, larastan:PHPStan\Rules\Functions\ParameterCastableToNumberRule |
| Should | Omit redundant asc arguments in conventional query ordering | shift:workbench/streamline-order-methods |
| Should | Use ScopedBy for equivalent class-based global scope registration when adopting model attributes | shift:workbench/laravel-global-scope-attribute |
| Should | Count matching rows in SQL instead of hydrating models to count them | partial: larastan:NoUnnecessaryCollectionCallRule |
| Should | Use enum backing values as PHP array keys | partial: larastan:PHPStan\Rules\Arrays\InvalidKeyInArrayDimFetchRule, partial: larastan:PHPStan\Rules\Arrays\InvalidKeyInArrayItemRule |
| Should | Annotate custom builders and their model registration with matching generic types | partial: larastan:PHPStan\Rules\Generics\ClassAncestorsRule, partial: larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule |
| Should | Import Blade View and Vite facades explicitly | rector:RenameClassRector |
| Should | Replace obsolete view expression types | rector:RenameClassRector |
| Should | Enforce agreed formatting with automated tooling | partial: pint:Pint/laravel_blade |
| Should | Indent each nested array element by one indentation level | pint:array_indentation |
| Should | Declare arrays with short [] syntax | pint:array_syntax, shift:workbench/adopt-short-arrays |
| Should | Put one space on each side of binary operators | pint:binary_operator_spaces |
| Should | Place PHP code on a line after the opening tag and separate it with a blank line | pint:blank_line_after_opening_tag |
| Should | Put a blank line before return and continue statements where a preceding statement exists | pint:blank_line_before_statement |
| Should | Separate groups of constant, class, and function imports with blank lines | pint:blank_line_between_import_groups |
| Should | Use single spaces in class declarations and keep single-item inheritance lists on one line | pint:class_definition |
| Should | Write nullable type declarations without spaces between ? and the type | pint:compact_nullable_type_declaration |
| Should | Write true, false, and null in lowercase | pint:constant_case |
| Should | Enclose every control-structure body in braces | pint:control_structure_braces |
| Should | Use elseif instead of else if | pint:elseif |
| Should | Save PHP files as UTF-8 without a byte-order mark | pint:encoding |
| Should | Use <?php or <?= instead of other PHP opening-tag variants | pint:full_opening_tag |
| Should | Normalize function declaration spacing, including one space before a closure use clause | pint:function_declaration |
| Should | Normalize legacy @inheritDocs tags to @inheritDoc with an explicit PHPDoc tag replacement map | pint:general_phpdoc_tag_rename |
| Should | Use nowdoc instead of heredoc when the string needs no interpolation | pint:heredoc_to_nowdoc |
| Should | Separate include and require from their paths with one space and remove unnecessary path parentheses | pint:include |
| Should | Use post-increment and post-decrement when the expression result is unused | pint:increment_style |
| Should | Indent PHP code with spaces instead of tabs | pint:indentation_type |
| Should | Use lowercase numeric base prefixes and uppercase hexadecimal digits in integer literals | pint:integer_literal_case |
| Should | Remove unused variables from closure use lists | pint:lambda_not_used_import |
| Should | Use LF line endings consistently in PHP files | pint:line_ending |
| Should | Put code after a PHP opening tag on a new line | pint:linebreak_after_opening_tag |
| Should | Write PHP keywords in lowercase | pint:lowercase_keywords |
| Should | Write self, static, and parent in lowercase | pint:lowercase_static_reference |
| Should | Use the canonical uppercase spelling of PHP magic constants | pint:magic_constant_casing |
| Should | Remove spaces before argument commas and put one space after commas without forcing multiline argument lists to expand | pint:method_argument_space |
| Should | Call native PHP functions using their canonical casing | pint:native_function_casing |
| Should | Call canonical PHP functions instead of their aliases | pint:no_alias_functions |
| Should | Use canonical language constructs such as exit instead of aliases such as die | pint:no_alias_language_construct_call |
| Should | Remove redundant b or B prefixes from string literals | pint:no_binary_string |
| Should | Do not put blank lines immediately after a class opening brace | pint:no_blank_lines_after_class_opening |
| Should | Place PHPDoc blocks directly above the documented element without an intervening blank line | pint:no_blank_lines_after_phpdoc |
| Should | Remove empty PHPDoc blocks | pint:no_empty_phpdoc |
| Should | Remove redundant empty semicolon statements | pint:no_empty_statement |
| Should | Remove repeated blank lines and extra blank lines associated with throw statements and imports | pint:no_extra_blank_lines |
| Should | Do not split an array key and its => operator across lines | pint:no_multiline_whitespace_around_double_arrow |
| Should | Put each PHP statement on its own line | pint:no_multiple_statements_per_line |
| Should | Remove spaces immediately before statement-ending semicolons | pint:no_singleline_whitespace_before_semicolons |
| Should | Do not put spaces immediately inside or outside array-offset brackets | pint:no_spaces_around_offset |
| Should | Do not add trailing commas to lists written entirely on one line | pint:no_trailing_comma_in_singleline |
| Should | Remove trailing whitespace from non-blank code lines | pint:no_trailing_whitespace |
| Should | Remove trailing whitespace from comment and PHPDoc lines | pint:no_trailing_whitespace_in_comment |
| Should | Remove unnecessary parentheses around break, clone, continue, echo, return, case, and yield expressions | pint:no_unneeded_control_parentheses |
| Should | Remove superfluous braces while retaining control-structure body braces | pint:no_unneeded_braces |
| Should | Do not declare optional parameters before required parameters | pint:no_unreachable_default_argument_value |
| Should | Remove empty return statements at the end of functions | pint:no_useless_return |
| Should | Do not put whitespace before array-item commas | pint:no_whitespace_before_comma_in_array |
| Should | Remove whitespace from blank lines | pint:no_whitespace_in_blank_line |
| Should | Use ?Type syntax for nullable single-type declarations | pint:nullable_type_declaration |
| Should | Do not put spaces around -> or ?-> | pint:object_operator_without_whitespace |
| Should | Sort imports alphabetically within groups ordered as constants, classes, then functions | pint:ordered_imports |
| Should | Left-align PHPDoc tag contents and use two spaces after @param | pint:phpdoc_align |
| Should | Indent PHPDoc blocks to the same level as the documented element | pint:phpdoc_indent |
| Should | Normalize the braces and syntax of inline PHPDoc tags | pint:phpdoc_inline_tag_normalizer |
| Should | Remove obsolete @access PHPDoc tags | pint:phpdoc_no_access |
| Should | Order @param, @return, and @throws PHPDoc tags in that sequence | pint:phpdoc_order |
| Should | Use int, bool, and float instead of integer, boolean, real, or double in PHPDoc | pint:phpdoc_scalar |
| Should | Use consistent spaces in single-line @var PHPDoc comments | pint:phpdoc_single_line_var_spacing |
| Should | Remove empty content lines at the start and end of PHPDoc blocks | pint:phpdoc_trim |
| Should | Use the canonical casing of built-in PHPDoc types | pint:phpdoc_types |
| Should | Put no space before and one space after the return-type colon | pint:return_type_declaration |
| Should | End PHP files without closing tags with exactly one newline | pint:single_blank_line_at_eof |
| Should | Declare each class property and constant in its own statement | pint:single_class_element_per_statement |
| Should | Use a separate use statement for every import | pint:single_import_per_statement |
| Should | Put each import on its own line and one blank line after the import block | pint:single_line_after_imports |
| Should | Put one space between a single-line comment marker and its text | pint:single_line_comment_spacing |
| Should | Use // instead of # for single-line comments | pint:single_line_comment_style |
| Should | Write empty class and function bodies as {} on the same line when permitted by the brace rules | pint:single_line_empty_body |
| Should | Use single quotes for strings that need no interpolation or escaping available only in double quotes | pint:single_quote |
| Should | Put one space after semicolons when another expression follows on the same line | pint:space_after_semicolon |
| Should | Remove spaces immediately inside parentheses | pint:spaces_inside_parentheses |
| Should | Use != instead of <> | pint:standardize_not_equals |
| Should | Indent statements according to their nesting level | pint:statement_indentation |
| Should | Put one space around ternary operators | pint:ternary_operator_spaces |
| Should | Add a trailing comma to multiline arrays | pint:trailing_comma_in_multiline |
| Should | Remove spaces immediately inside single-line array brackets | pint:trim_array_spaces |
| Should | Put one space between a type declaration and its variable | pint:type_declaration_spaces |
| Should | Write union and intersection type operators without surrounding spaces | pint:types_spaces |
| Should | Put one space after array-item commas when the next item is on the same line | pint:whitespace_after_comma_in_array |
| Should | Read and write only accessible declared properties or properties supported by a documented dynamic API | larastan:PHPStan\Rules\Properties\AccessPropertiesInAssignRule, larastan:PHPStan\Rules\Properties\AccessPropertiesRule |
| Should | Use the injected request object in controllers and middleware instead of equivalent request facade or helper calls | shift:workbench/leverage-request-injection |
| Should | Import Laravel facade classes explicitly instead of relying on global aliases | rector:RenameClassRector, shift:workbench/replace-global-facades |
| Should | Memoize derived values with Livewire computed methods | partial: rector:LivewireComponentComputedMethodToComputedAttributeRector |
| Should | Lint PHP syntax in CI | partial: shift:public/ci/syntax |
| Should | Verify required extensions before moving to Cloud | partial: shift:public/cloud/extensions |
| Should | Use Laravel-13-compatible dependency constraints | partial: shift:workbench/bump-laravel-dependencies |
| Should | Use canonical PHP function names | partial: pint:no_alias_functions |
| Should | Avoid redundant value wrappers | larastan:NoUselessValueFunctionCallsRule, rector:RemoveRedundantValueCallsRector |
| Should | Format fluent chains consistently | partial: pint:method_chaining_indentation |
| Should | Use synchronous dispatch explicitly for synchronous commands | rector:DispatchNonShouldQueueToDispatchSyncRector |
| Should | Keep backed enums valid | larastan:PHPStan\Rules\Classes\EnumSanityRule |
| Should | Use unique parameter names | larastan:PHPStan\Rules\Functions\RedefinedParametersRule |
| Should | Reference Laravel 13 request-forgery middleware in code and tests | partial: rector:RenameClassRector |
| Should | Inject services when adopting dependency injection over facades | partial: rector:ArgumentFuncCallToMethodCallRector, partial: rector:StaticCallToMethodCallRector |
| Should | Use null coalescing for equivalent isset fallbacks | pint:ternary_to_null_coalescing |
| Should | Do not declare a class constant named class | larastan:PHPStan\Rules\Constants\ClassAsClassConstantRule |
| Should | Omit final on private constants and ordinary private methods | larastan:PHPStan\Rules\Constants\FinalPrivateConstantRule, larastan:PHPStan\Rules\Methods\FinalPrivateMethodRule |
| Should | Avoid reserved parameter variable names | larastan:PHPStan\Rules\Functions\InvalidParameterNameRule |
| Should | Leave constructor return types undeclared | larastan:PHPStan\Rules\Methods\ConstructorReturnTypeRule |
| Should | Put literals first under an adopted Yoda comparison policy | pint:yoda_style |
| Should | Respect internal API boundaries | partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalClassConstantUsageExtension, partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalClassNameUsageExtension, partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalFunctionUsageExtension, partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalMethodUsageExtension, partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalPropertyUsageExtension |
| Should | Write useful PHPDoc descriptions as sentences | partial: pint:phpdoc_summary |
| Should | Leave one blank line after namespaces | pint:blank_line_after_namespace |
| Should | Separate the namespace from the file preamble | pint:blank_lines_before_namespace |
| Should | Use the selected brace placement | pint:braces_position |
| Should | Keep namespace names clean and unindented | pint:clean_namespace, pint:no_leading_namespace_whitespace |
| Should | Join continuation keywords to closing braces | pint:control_structure_continuation_position |
| Should | Use canonical PHP magic method casing | pint:magic_method_casing |
| Should | Keep semicolons with their expression | pint:multiline_whitespace_before_semicolons |
| Should | Use canonical lowercase native types | pint:native_type_declaration_casing |
| Should | Omit leading separators from imports | pint:no_leading_import_slash |
| Should | Attach call parentheses to the callee | pint:no_spaces_after_function_name |
| Should | Remove redundant import aliases | pint:no_unneeded_import_alias |
| Should | Order interface lists consistently | pint:ordered_interfaces |
| Should | Omit redundant property names from type tags | pint:phpdoc_var_without_name |
| Should | Order member modifiers consistently | partial: pint:modifier_keywords |
| Should | Keep variables on the left of comparisons | pint:yoda_style |
| Should | Import one trait per statement | pint:single_trait_insert_per_statement |
| Should | Use canonical internal class casing | pint:class_reference_name_casing |
| Should | Give intentional empty loops braces | pint:empty_loop_body |
| Should | Remove redundant empty-string concatenation | pint:no_useless_concat_operator |
| Should | Collapse extra PHPDoc blank lines | pint:phpdoc_trim_consecutive_blank_line_separation |
| Should | Put annotation types before variable names | pint:phpdoc_var_annotation_correct_order |
| Should | Separate return and control statements under the custom policy | pint:blank_line_before_statement |
| Should | Destructure only offset-accessible values | larastan:PHPStan\Rules\Arrays\ArrayDestructuringRule |
| Should | Remove loops over provably empty input | larastan:PHPStan\Rules\Arrays\DeadForeachRule |
| Should | Avoid duplicate array literal keys | larastan:PHPStan\Rules\Arrays\DuplicateKeysInLiteralArraysRule |
| Should | Use supported array key types | larastan:PHPStan\Rules\Arrays\InvalidKeyInArrayDimFetchRule, larastan:PHPStan\Rules\Arrays\InvalidKeyInArrayItemRule |
| Should | Iterate only iterable values | larastan:PHPStan\Rules\Arrays\IterableInForeachRule |
| Should | Read existing supported offsets | larastan:PHPStan\Rules\Arrays\OffsetAccessWithoutDimForReadingRule, larastan:PHPStan\Rules\Arrays\NonexistentOffsetInArrayDimFetchRule |
| Should | Write offsets only on writable receivers | larastan:PHPStan\Rules\Arrays\OffsetAccessAssignmentRule |
| Should | Preserve array and ArrayAccess element types | larastan:PHPStan\Rules\Arrays\OffsetAccessValueAssignmentRule |
| Should | Unpack only iterable values into arrays | larastan:PHPStan\Rules\Arrays\UnpackIterableInArrayRule |
| Should | Validate attribute targets and arguments | larastan:PHPStan\Rules\Classes\ClassAttributesRule, larastan:PHPStan\Rules\Classes\ClassConstantAttributesRule, larastan:PHPStan\Rules\EnumCases\EnumCaseAttributesRule, larastan:PHPStan\Rules\Functions\ArrowFunctionAttributesRule, larastan:PHPStan\Rules\Functions\ClosureAttributesRule, larastan:PHPStan\Rules\Functions\FunctionAttributesRule |
| Should | Reference existing accessible constants | larastan:PHPStan\Rules\Classes\ClassConstantRule, larastan:PHPStan\Rules\Constants\ConstantRule |
| Should | Declare each symbol once in its scope | larastan:PHPStan\Rules\Classes\DuplicateDeclarationRule, larastan:PHPStan\Rules\Names\UsedNamesRule, larastan:PHPStan\Rules\Classes\DuplicateClassDeclarationRule, larastan:PHPStan\Rules\Functions\DuplicateFunctionDeclarationRule |
| Should | Extend existing extensible classes | larastan:PHPStan\Rules\Classes\ExistingClassInClassExtendsRule |
| Should | Use real object types in instanceof | larastan:PHPStan\Rules\Classes\ExistingClassInInstanceOfRule |
| Should | Make custom attributes constructible | larastan:PHPStan\Rules\Classes\NonClassAttributeClassRule |
| Should | Declare and satisfy trait interface requirements | larastan:PHPStan\Rules\Classes\RequireImplementsRule, larastan:PHPStan\Rules\PhpDoc\RequireImplementsDefinitionClassRule, larastan:PHPStan\Rules\PhpDoc\RequireImplementsDefinitionTraitRule |
| Should | Remove genuinely unused parameters | partial: larastan:PHPStan\Rules\Classes\UnusedConstructorParametersRule, partial: larastan:PHPStan\Rules\Functions\UnusedFunctionParametersRule, partial: larastan:PHPStan\Rules\Methods\UnusedMethodParametersRule |
| Should | Remove statically impossible or redundant branches | partial: larastan:PHPStan\Rules\Variables\EmptyRule, partial: larastan:PHPStan\Rules\Variables\IssetRule, partial: larastan:PHPStan\Rules\Variables\NullCoalesceRule, partial: larastan:PHPStan\Rules\Classes\ImpossibleInstanceOfRule, partial: larastan:PHPStan\Rules\Comparison\BooleanAndConstantConditionRule, partial: larastan:PHPStan\Rules\Comparison\BooleanNotConstantConditionRule, partial: larastan:PHPStan\Rules\Comparison\ImpossibleCheckTypeMethodCallRule, partial: larastan:PHPStan\Rules\Comparison\ImpossibleCheckTypeFunctionCallRule, partial: larastan:PHPStan\Rules\Comparison\ImpossibleCheckTypeStaticMethodCallRule |
| Should | Keep match arms exhaustive and reachable | larastan:PHPStan\Rules\Comparison\MatchExpressionRule |
| Should | Make unbounded loops intentional | partial: larastan:PHPStan\Rules\Comparison\DoWhileLoopConstantConditionRule, partial: larastan:PHPStan\Rules\Comparison\WhileLoopAlwaysTrueConditionRule |
| Should | Keep constant annotations precise and compatible | larastan:PHPStan\Rules\PhpDoc\IncompatibleClassConstantPhpDocTypeRule, larastan:PHPStan\Rules\Constants\MissingClassConstantTypehintRule |
| Should | Preserve inherited and composed constant contracts | larastan:PHPStan\Rules\Constants\OverridingConstantRule, larastan:PHPStan\Rules\Traits\ConflictingTraitConstantsRule |
| Should | Use results of pure operations | larastan:PHPStan\Rules\DeadCode\CallToConstructorStatementWithoutImpurePointsRule, larastan:PHPStan\Rules\DeadCode\CallToFunctionStatementWithoutImpurePointsRule, larastan:PHPStan\Rules\DeadCode\CallToMethodStatementWithoutImpurePointsRule, larastan:PHPStan\Rules\DeadCode\CallToStaticMethodStatementWithoutImpurePointsRule, larastan:PHPStan\Rules\DeadCode\NoopRule, larastan:PHPStan\Rules\Functions\CallToFunctionStatementWithoutSideEffectsRule |
| Should | Remove unreachable statements after exits | larastan:PHPStan\Rules\DeadCode\UnreachableStatementRule |
| Should | Remove unconsumed private members | partial: larastan:PHPStan\Rules\DeadCode\UnusedPrivateConstantRule, partial: larastan:PHPStan\Rules\DeadCode\UnusedPrivateMethodRule, partial: larastan:PHPStan\Rules\DeadCode\UnusedPrivatePropertyRule |
| Should | Keep catch clauses reachable | partial: larastan:PHPStan\Rules\Exceptions\CatchWithUnthrownExceptionRule |
| Should | Do not override exits in finally | larastan:PHPStan\Rules\Exceptions\OverwrittenExitPointByFinallyRule, larastan:PHPStan\Rules\Functions\ReturnTypeAfterFinallyRule |
| Should | Remove no-op callback-free array filters | larastan:PHPStan\Rules\Functions\ArrayFilterRule |
| Should | Avoid reindexing known lists | larastan:PHPStan\Rules\Functions\ArrayValuesRule |
| Should | Return values matching native contracts | larastan:PHPStan\Rules\Missing\MissingReturnRule, larastan:PHPStan\Rules\Functions\ArrowFunctionReturnTypeRule, larastan:PHPStan\Rules\Functions\ClosureReturnTypeRule, larastan:PHPStan\Rules\Functions\ReturnTypeAfterFinallyRule, larastan:PHPStan\Rules\Functions\ReturnTypeRule, larastan:PHPStan\Rules\Methods\ReturnTypeRule |
| Should | Do not consume void expression results | partial: larastan:PHPStan\Rules\Comparison\UsageOfVoidMatchExpressionRule |
| Should | Call accessible symbols with compatible arguments | larastan:PHPStan\Rules\Classes\InstantiationRule, larastan:PHPStan\Rules\Functions\CallToFunctionParametersRule, larastan:PHPStan\Rules\Functions\DefineParametersRule, larastan:PHPStan\Rules\Methods\CallMethodsRule, larastan:PHPStan\Rules\Methods\CallStaticMethodsRule, larastan:PHPStan\Rules\Functions\CallCallablesRule |
| Should | Create first-class callables from real callable symbols | larastan:PHPStan\Rules\Classes\InstantiationCallableRule, larastan:PHPStan\Rules\Functions\FunctionCallableRule, larastan:PHPStan\Rules\Methods\MethodCallableRule, larastan:PHPStan\Rules\Methods\StaticMethodCallableRule |
| Should | Use resolvable declaration types | larastan:PHPStan\Rules\Functions\ExistingClassesInArrowFunctionTypehintsRule, larastan:PHPStan\Rules\Functions\ExistingClassesInClosureTypehintsRule, larastan:PHPStan\Rules\Functions\ExistingClassesInTypehintsRule, larastan:PHPStan\Rules\Methods\ExistingClassesInTypehintsRule, larastan:PHPStan\Rules\Properties\ExistingClassesInPropertiesRule, larastan:PHPStan\Rules\Properties\ExistingClassesInPropertyHookTypehintsRule |
| Should | Match defaults to parameter types | larastan:PHPStan\Rules\Functions\IncompatibleArrowFunctionDefaultParameterTypeRule, larastan:PHPStan\Rules\Functions\IncompatibleClosureDefaultParameterTypeRule, larastan:PHPStan\Rules\Functions\IncompatibleDefaultParameterTypeRule, larastan:PHPStan\Rules\Methods\IncompatibleDefaultParameterTypeRule |
| Should | Choose one filter failure mode | larastan:PHPStan\Rules\Functions\FilterVarRule |
| Should | Avoid nested named functions | larastan:PHPStan\Rules\Functions\InnerFunctionRule |
| Should | Capture only needed valid closure variables | larastan:PHPStan\Rules\Functions\InvalidLexicalVariablesInClosureUseRule, larastan:PHPStan\Rules\Functions\UnusedClosureUsesRule |
| Should | Match formatted-output placeholders and arguments | larastan:PHPStan\Rules\Functions\PrintfArrayParametersRule, larastan:PHPStan\Rules\Functions\PrintfParametersRule, larastan:PHPStan\Rules\Functions\PrintfParameterTypeRule |
| Should | Validate random integer bounds | partial: larastan:PHPStan\Rules\Functions\RandomIntParametersRule |
| Should | Read only declared console inputs | larastan:UndefinedArgumentOrOptionRule |
| Should | Gate newer PHP syntax on the supported runtime floor | partial: larastan:PHPStan\Rules\Constants\ConstantAttributesRule, partial: larastan:PHPStan\Rules\Properties\PropertyInClassRule |
| Should | Throw exception objects | partial: larastan:PHPStan\Rules\Exceptions\ThrowExprTypeRule |
| Should | Use declared function-name casing | larastan:PHPStan\Rules\Functions\CallToNonExistentFunctionRule |
| Should | Keep enum declarations nongeneric | larastan:PHPStan\Rules\Generics\EnumTemplateTypeRule |
| Should | Resolve statically included files | partial: larastan:PHPStan\Rules\Keywords\RequireFileExistsRule |
| Should | Avoid unstable named arguments across inheritance | partial: larastan:PHPStan\Rules\Methods\MethodCallWithPossiblyRenamedNamedArgumentRule |
| Should | Preserve inherited method contracts | larastan:PHPStan\Rules\Methods\OverridingMethodRule, larastan:PHPStan\Rules\Methods\MethodSignatureRule |
| Should | Import real functions and constants with correct names | larastan:PHPStan\Rules\Namespaces\ExistingNamesInGroupUseRule, larastan:PHPStan\Rules\Namespaces\ExistingNamesInUseRule |
| Should | Use operators with compatible operand types | larastan:PHPStan\Rules\Operators\InvalidIncDecOperationRule, larastan:PHPStan\Rules\Operators\InvalidBinaryOperationRule, larastan:PHPStan\Rules\Operators\InvalidComparisonOperationRule, larastan:PHPStan\Rules\Operators\InvalidUnaryOperationRule |
| Should | Express argument-dependent return types accurately | partial: larastan:PHPStan\Rules\PhpDoc\FunctionConditionalReturnTypeRule, partial: larastan:PHPStan\Rules\PhpDoc\MethodConditionalReturnTypeRule |
| Should | Keep PHPDoc compatible with native declarations | larastan:PHPStan\Rules\PhpDoc\IncompatiblePhpDocTypeRule, larastan:PHPStan\Rules\PhpDoc\IncompatiblePropertyHookPhpDocTypeRule, larastan:PHPStan\Rules\PhpDoc\IncompatiblePropertyPhpDocTypeRule |
| Should | Use valid recognized PHPStan PHPDoc tags | larastan:PHPStan\Rules\PhpDoc\InvalidPHPStanDocTagRule, larastan:PHPStan\Rules\PhpDoc\InvalidPhpDocTagValueRule |
| Should | Document throwable exception types | larastan:PHPStan\Rules\PhpDoc\InvalidThrowsPhpDocValueRule |
| Should | Do not overwrite inferred types with contradictory assertions | larastan:PHPStan\Rules\PhpDoc\VarTagChangedExpressionTypeRule |
| Should | Access declared accessible static properties | larastan:PHPStan\Rules\Properties\AccessStaticPropertiesInAssignRule, larastan:PHPStan\Rules\Properties\AccessStaticPropertiesRule |
| Should | Assign values matching writable property types | larastan:PHPStan\Rules\Properties\DefaultValueTypesAssignedToPropertiesRule, larastan:PHPStan\Rules\Properties\TypesAssignedToPropertiesRule |
| Should | Preserve inherited property contracts | larastan:PHPStan\Rules\Properties\OverridingPropertyRule, larastan:PHPStan\Rules\Properties\ReadOnlyPropertyAssignRule |
| Should | Escape regex data with the actual delimiter | larastan:PHPStan\Rules\Regexp\RegularExpressionQuotingRule |
| Should | Remove unneeded return union members | larastan:PHPStan\Rules\TooWideTypehints\TooWideArrowFunctionReturnTypehintRule, larastan:PHPStan\Rules\TooWideTypehints\TooWideClosureReturnTypehintRule, larastan:PHPStan\Rules\TooWideTypehints\TooWideFunctionReturnTypehintRule, larastan:PHPStan\Rules\TooWideTypehints\TooWideMethodReturnTypehintRule |
| Should | Keep closed property types precise | larastan:PHPStan\Rules\TooWideTypehints\TooWidePropertyTypeRule |
| Should | Analyze traits through consuming classes | partial: larastan:PHPStan\Rules\Traits\NotAnalysedTraitRule |
| Should | Define local variables on every read path | larastan:PHPStan\Rules\Variables\CompactVariablesRule, larastan:PHPStan\Rules\Variables\DefinedVariableRule |
| Should | Unset only valid mutable targets | larastan:PHPStan\Rules\Variables\UnsetRule |
| Should | Clone only cloneable objects | larastan:PHPStan\Rules\Variables\VariableCloningRule |
| Should | Remove accidental output around PHP files | larastan:PHPStan\Rules\Whitespace\FileWhitespaceRule, pint:encoding, pint:no_closing_tag |
| Should | Narrow unions before member-specific operations | partial: larastan:PHPStan\Rules\Methods\CallMethodsRule, partial: larastan:PHPStan\Rules\Properties\AccessPropertiesRule, partial: larastan:PHPStan\Rules\Functions\CallToFunctionParametersRule |
| Should | Handle nullable receivers before dereferencing | larastan:PHPStan\Rules\Methods\CallMethodsRule, larastan:PHPStan\Rules\Properties\AccessPropertiesRule |
| Should | Narrow mixed values before typed operations | partial: larastan:PHPStan\Rules\Functions\CallToFunctionParametersRule, partial: larastan:PHPStan\Rules\Methods\CallMethodsRule, partial: larastan:PHPStan\Rules\Properties\AccessPropertiesRule |
| Should | Remove impossible strict membership candidates | larastan:PHPStan\Rules\Comparison\ImpossibleInArrayHaystackFiniteTypesRule |
| Should | Remove writes whose values are never read | larastan:PHPStan\Rules\DeadCode\UnusedVariableRule |
| Should | Remove impossible throws annotations | partial: larastan:PHPStan\Rules\Exceptions\TooWideFunctionThrowTypeRule, partial: larastan:PHPStan\Rules\Exceptions\TooWideMethodThrowTypeRule, partial: larastan:PHPStan\Rules\Exceptions\TooWidePropertyHookThrowTypeRule |
| Should | Initialize typed properties before use | partial: larastan:PHPStan\Rules\Properties\UninitializedPropertyRule |
| Should | Remove provably redundant sorts | partial: larastan:PHPStan\Rules\Functions\SortWithoutEffectRule |
| Should | Use strict comparisons when types matter | pint:strict_comparison |
| Should | Enable class-based model factory discovery | rector:AddHasFactoryToModelsRector |
| Should | Use canonical PHPUnit lifecycle signatures | pint:php_unit_set_up_tear_down_visibility, shift:workbench/fix-phpunit-methods |
| Should | Start compatible headless browser infrastructure in CI | partial: shift:ci-generator/dusk-job |
| Should | Import the Validator facade explicitly | rector:RenameClassRector |
| Should | Use the selected local filesystem abstraction consistently | rector:UnlinkFuncCallToFileFacadeDeleteRector |
| Should | Reconcile published configuration with installed versions during upgrades | partial: shift:public/laravel13/config-env |
| Should | Resolve the current container lazily instead of capturing the boot-time container in an Octane singleton | partial: larastan:OctaneCompatibilityRule |
| Should | Pass only array-valued configuration keys to Config::collection() | larastan:ConfigCollectionRule |
| Should | Use the Date facade for date creation when the application relies on Laravel’s configurable date implementation | rector:CarbonToDateFacadeRector |
| Should | Use explicit container resolution, binding, and existence methods when adopting the explicit container API style | rector:ArrayDimFetchToMethodCallRector |
| Should | Replace removed spatial column helpers with supported geometry definitions | rector:RefactorBlueprintGeometryColumnsRector |
| Should | Use current Livewire component formats for new projects | partial: shift:livewire-mfc-converter |
| Should | Use explicit date-intent query helpers | rector:DateWhereClauseToShorthandRector |
| Should | Reference only relationships that are defined and statically checked | larastan:RelationExistenceRule |
| Should | Declare void on methods that return no value | partial: rector:AddVoidReturnTypeWhereNoReturnRector, partial: larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule, partial: larastan:PHPStan\Rules\Methods\ReturnTypeRule |
| Should | Invoke local scopes through a query builder and provide analysis metadata where required | partial: larastan:NoImplicitQueryBuilderCallRule, partial: rector:AddGenericBuilderToScopesRector |
| Should | Read query-expression SQL with getValue() and the connection grammar | rector:DatabaseExpressionCastsToMethodCallRector, rector:DatabaseExpressionToStringToMethodCallRector |
| Should | Type relationship query-constraint callbacks with the appropriate builder | rector:EloquentWhereRelationTypeHintingParameterRector |
| Should | Match the installed framework method signature in custom implementations and overrides | partial: rector:ArgumentAdderRector, partial: larastan:PHPStan\Rules\Methods\OverridingMethodRule |
| Should | Replace removed Cashier stripeOptions() overrides with the supported stripe() extension point | rector:CashierStripeOptionsToStripeRector |
| Should | Indent Blade with four spaces | partial: pint:Pint/laravel_blade |
| Should | Replace Volt base classes only when migrating away from Volt | partial: rector:RenameClassRector, partial: shift:livewire-4/volt-to-sfc |
| Should | Pass exception classes to conditional throw helpers | rector:ThrowIfAndThrowUnlessExceptionsToUseClassStringRector |
| Should | Confirm dynamic usage before removing unused views | partial: larastan:UnusedViewsRule |
| Should | Review long-lived container bindings when adopting Octane | partial: larastan:OctaneCompatibilityRule |
| Should | Use Hidden attributes for command visibility when adopting Laravel 13 attributes | rector:CommandHiddenPropertyToHiddenAttributeRector |
| Should | Avoid terminating requests while debugging session writes | partial: rector:RemoveDumpDataDeadCodeRector |
| Should | Remove obsolete translation roots after migration | partial: shift:upgrade-9/lang-folder |
| Should | Put intentionally cacheable pages in a dedicated stateless middleware group | shift:public/fast-laravel/static-group |
| Should | Use fluent route configuration methods instead of legacy array options | shift:workbench/adopt-fluent-routes |
| Should | Format comments consistently | pint:single_line_comment_spacing, pint:align_multiline_comment |
| Should | Prefer native declarations over redundant PHPDoc | partial: pint:no_superfluous_phpdoc_tags, partial: shift:public/fixer/redundant-docblock-types |
| Should | Match queued dispatch arguments to constructors | larastan:CheckDispatchArgumentTypesCompatibleWithClassConstructorRule, larastan:PHPStan\Rules\Methods\MissingMethodParameterTypehintRule |
| Should | Adopt level six for complete type declarations | partial: larastan:PHPStan\Rules\Functions\MissingFunctionParameterTypehintRule, partial: larastan:PHPStan\Rules\Functions\MissingFunctionReturnTypehintRule, partial: larastan:PHPStan\Rules\Methods\MissingMethodParameterTypehintRule, partial: larastan:PHPStan\Rules\Properties\MissingPropertyTypehintRule, partial: larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule |
| Should | Make classes final unless designed for extension | partial: shift:phpunit/final-test-classes |
| Should | Import classes and globals under the import policy | pint:global_namespace_import, pint:fully_qualified_strict_types |
| Should | Space concatenation operators under the custom policy | pint:concat_space |
| Should | Use all() when recursive conversion is unnecessary | larastan:NoUnnecessaryEnumerableToArrayCallsRule, rector:ConvertEnumerableToArrayToAllRector |
| Should | Use strict closures for null or typed view data | partial: pint:strict_comparison |
| Should | Migrate Volt classes when adopting Livewire 4 built-in components | partial: shift:livewire-4/volt-to-sfc |
| Should | Type-hint dispatched-object assertion predicates | partial: rector:AssertWithClassStringToTypeHintedClosureRector |
| Should | Replace deprecated Livewire Rule attributes | rector:RenameAttributeRector |
| Consider | Replace abandoned dependencies | partial: shift:preshift/abandoned-dependencies |
| Consider | Gate interface property hooks by the PHP target | larastan:PHPStan\Rules\Properties\PropertiesInInterfaceRule |
| Consider | Accept Throwable in custom exception handlers | rector:AddParamTypeDeclarationRector |
| Consider | Keep exception-handler ignore public | rector:ChangeMethodVisibilityRector |
| Consider | Qualify global PHP attributes in framework-style code | pint:global_namespace_import |
| Consider | Apply the repository’s selected helper or facade idiom consistently for equivalent operations | partial: rector:HelperFuncCallToFacadeClassRector, partial: shift:workbench/adopt-common-helpers |
| Consider | Define every statically referenced file-based translation key in the configured translation files | larastan:NoMissingTranslationsRule |
| Consider | Keep constant values compatible with declared and static-analysis-configured constant types | larastan:PHPStan\Rules\Constants\ValueAssignedToDefineRule, larastan:PHPStan\Rules\Constants\ValueAssignedToGlobalConstantRule, larastan:PHPStan\Rules\Constants\ValueAssignedToClassConstantRule |
| Consider | Initialize readonly properties only through write paths allowed by the supported PHP version | larastan:PHPStan\Rules\Properties\ReadOnlyPropertyAssignRule |
| Consider | Use a fresh binding or current-container resolver for configuration in long-lived Octane services | partial: larastan:OctaneCompatibilityRule |
| Consider | Adopt an explicit forward-only migration policy | partial: shift:workbench/remove-down-migration |
| Consider | Implement the user-provider contract for custom user storage | partial: larastan:PHPStan\Rules\Methods\MissingMethodImplementationRule, partial: larastan:PHPStan\Rules\Methods\OverridingMethodRule |
| Consider | Omit unused exception bindings from catch clauses | rector:RemoveUnusedVariableInCatchRector |
| Consider | Declare only available relationships in model eager-load defaults | larastan:ModelRelationDefaultsRule |
| Consider | Import Laravel classes explicitly instead of relying on global aliases | rector:RenameClassRector |
| Consider | Migrate custom mail integrations from SwiftMailer to Symfony Mailer | partial: rector:RenameMethodRector, partial: shift:upgrade-9/swiftmailer |
| Consider | Preserve parent event-provider registration | rector:AddParentRegisterToEventServiceProviderRector |
| Consider | Import applied PHP attribute classes | partial: pint:global_namespace_import, partial: pint:fully_qualified_strict_types |
| Consider | Type method parameters explicitly | partial: larastan:PHPStan\Rules\Methods\MissingMethodParameterTypehintRule, partial: shift:workbench/type-hints |
| Consider | Use middlewareAliases in a retained legacy HTTP kernel | partial: rector:RenamePropertyRector |
| Consider | Write declare directive assignments without spaces around the equals sign | pint:declare_equal_normalize |
| Consider | Write declare(...) without spaces around its parentheses | pint:declare_parentheses |
| Consider | Remove PHPDoc types that repeat native declarations | pint:no_superfluous_phpdoc_tags |
| Consider | End switch case labels with a colon instead of a semicolon | pint:switch_case_semicolon_to_colon |
| Consider | Use one space between case and its value and no extra space before its colon | pint:switch_case_space |
| Consider | Track and retire deprecations before upgrades | partial: shift:public/laravel13/deprecations |
| Consider | Use Blade method-spoofing directives | shift:workbench/blade/method |
| Consider | Specify generic arguments on implemented contracts | larastan:PHPStan\Rules\Generics\ClassAncestorsRule |
| Consider | Replace removed synchronous dispatch APIs | rector:RenameFunctionRector, rector:RenameMethodRector |
| Consider | Import facades by their namespaced class | rector:RenameClassRector |
| Consider | Use bulk dispatch for independent untracked jobs | partial: larastan:NoBatchedUniqueJobRule |
| Consider | Implement the Laravel 13 bus dispatcher contract | larastan:PHPStan\Rules\Methods\MissingMethodImplementationRule, larastan:PHPStan\Rules\Methods\OverridingMethodRule |
| Consider | Reference actual model relationships in Eloquent relation queries | larastan:RelationExistenceRule |
| Consider | Key Composer CI caches by runtime and dependency definition | partial: shift:public/ci/composer-cache |
| Consider | Avoid removed PHPStan AST navigation attributes | larastan:PHPStan\Rules\Api\NodeConnectingVisitorAttributesRule |
| Consider | Use PHP-Parser node types supported by the analyzer | larastan:PHPStan\Rules\Api\OldPhpParser4ClassRule |
| Consider | Use PHPStan reflection for analyzed symbols | partial: larastan:PHPStan\Rules\Api\RuntimeReflectionFunctionRule, partial: larastan:PHPStan\Rules\Api\RuntimeReflectionInstantiationRule |
| Consider | Follow constructor-parenthesis requirements of the selected alternate preset | pint:new_with_parentheses |
| Consider | Use class-first import groups under the selected alternate preset | pint:ordered_imports |
| Consider | Import class names referenced in PHPDoc | pint:fully_qualified_strict_types |
| Consider | Mark intentional switch fall-through | pint:no_break_comment |
| Consider | Use compound offset updates only on compatible values | larastan:PHPStan\Rules\Arrays\OffsetAccessAssignOpRule |
| Consider | Resolve private static members through self | larastan:PHPStan\Rules\Classes\AccessPrivateConstantThroughStaticRule, larastan:PHPStan\Rules\Methods\CallPrivateMethodThroughStaticRule, larastan:PHPStan\Rules\Properties\AccessPrivatePropertyThroughStaticRule |
| Consider | Give mixins valid complete object types | larastan:PHPStan\Rules\Classes\MixinRule, larastan:PHPStan\Rules\Classes\MixinTraitRule, larastan:PHPStan\Rules\Classes\MixinTraitUseRule |
| Consider | Remove duplicate or impossible switch cases | larastan:PHPStan\Rules\Comparison\SwitchConditionRule |
| Consider | Validate date constructor strings | partial: larastan:PHPStan\Rules\DateTimeInstantiationRule |
| Consider | Request returned output before assigning it | partial: larastan:PHPStan\Rules\Functions\UselessFunctionReturnValueRule |
| Consider | Declare complete generator contracts | larastan:PHPStan\Rules\Generators\YieldInGeneratorRule, larastan:PHPStan\Rules\Generators\YieldTypeRule |
| Consider | Use backing storage in backed property hooks | larastan:PHPStan\Rules\Properties\GetNonVirtualPropertyHookReadRule, larastan:PHPStan\Rules\Properties\SetNonVirtualPropertyHookAssignRule |
| Consider | Honor native readonly property lifecycles | larastan:PHPStan\Rules\Properties\ReadOnlyPropertyRule, larastan:PHPStan\Rules\Properties\ReadOnlyPropertyAssignRefRule, larastan:PHPStan\Rules\Properties\MissingReadOnlyPropertyAssignRule |
| Consider | Respect readable and writable property directions | larastan:PHPStan\Rules\Properties\ReadingWriteOnlyPropertiesRule, larastan:PHPStan\Rules\Properties\WritingToReadOnlyPropertiesRule |
| Consider | Respect reference parameter input and output contracts | larastan:PHPStan\Rules\Variables\ParameterOutAssignedTypeRule, larastan:PHPStan\Rules\Variables\ParameterOutExecutionEndTypeRule |
| Consider | Remove unreferenced goto labels | larastan:PHPStan\Rules\Keywords\UnusedLabelRule |
| Consider | Break foreach reference aliases after iteration | partial: larastan:PHPStan\Rules\Variables\AssignToByRefExprFromForeachRule |
| Consider | Remove unused legacy application directories | partial: shift:public/linter/unused-directories |
| Consider | Keep classmap class identities unambiguous | partial: larastan:PHPStan\Rules\Classes\DuplicateClassDeclarationRule |
| Consider | Keep PHPStan assertion contracts truthful | larastan:PHPStan\Rules\PhpDoc\FunctionAssertRule, larastan:PHPStan\Rules\PhpDoc\MethodAssertRule |
| Consider | Call request validation on an injected request instance | rector:RequestStaticValidateToInjectRector |
| Consider | Replace obsolete validator failure methods | rector:RenameMethodRector |
| Consider | Replace obsolete message replacement methods | rector:RenameMethodRector |
| Consider | Reference the current upload-size middleware | rector:RenameClassRector |
| Consider | Sort trait imports | pint:ordered_traits |
| Consider | Do not promise inheritable private constructors | larastan:PHPStan\Rules\Methods\ConsistentConstructorDeclarationRule |
| Consider | Provide modern serialization methods with Serializable | larastan:PHPStan\Rules\Methods\MissingMagicSerializationMethodsRule |
| Consider | Check trait attribute targets and PHP support | larastan:PHPStan\Rules\Traits\TraitAttributesRule |
| Consider | Preserve application path method parameters | partial: rector:ArgumentAdderRector |
| Consider | Honor boolean results in custom cache write methods | partial: rector:AddReturnTypeDeclarationRector |
| Consider | Replace obsolete Cashier plan and cancellation method names | rector:RenameMethodRector |
| Consider | Use resolve(Service::class) for explicit fallback service resolution when adopting the resolve() idiom | rector:AppToResolveRector, shift:workbench/resolve-helper |
| Consider | Assign dynamically resolved services to explicitly typed locals before use when tooling cannot infer their type | rector:CallOnAppArrayAccessToStandaloneAssignRector |
| Consider | Apply the configured PHPDoc tag grouping and canonical tag conventions | pint:general_phpdoc_tag_rename, pint:phpdoc_separation, pint:phpdoc_return_self_reference, pint:phpdoc_no_alias_tag |
| Consider | Apply the selected formatter’s language-construct and unary-operator spacing | pint:single_space_around_construct, pint:unary_operator_spaces |
| Consider | Use explicit configuration repository methods when adopting an explicit repository API style | rector:ArrayDimFetchToMethodCallRector |
| Consider | Construct Collection explicitly when the repository chooses explicit class construction over the collect() helper | rector:FuncCallToNewRector |
| Consider | Migrate middleware, exception, routing, and provider configuration explicitly when adopting the streamlined skeleton | partial: shift:public/slimmer/bootstrap |
| Consider | Inject the database manager under an explicit dependency-injection policy | rector:StaticCallToMethodCallRector |
| Consider | Inject the schema builder under an explicit dependency-injection policy | rector:StaticCallToMethodCallRector |
| Consider | Resolve database services by supported class or contract | rector:ReplaceServiceContainerCallArgRector |
| Consider | Preserve provider behavior during application structure migration | partial: shift:public/slimmer/providers |
| Consider | Audit runtime dependencies before moving to Cloud | partial: shift:public/cloud/database, partial: shift:public/cloud/octane, partial: shift:public/cloud/php, partial: shift:public/cloud/system-commands, partial: shift:public/cloud/system-paths |
| Consider | Replace the removed Cashier removePaymentMethod API with deletePaymentMethod | partial: rector:RenameMethodRector |
| Consider | Use the void discard cast only as a standalone statement on PHP 8.5 or later | larastan:PHPStan\Rules\Cast\VoidCastRule |
| Consider | Consume NoDiscard results or explicitly discard them with supported syntax | larastan:PHPStan\Rules\Functions\CallToFunctionStatementWithNoDiscardRule, larastan:PHPStan\Rules\Methods\CallToMethodStatementWithNoDiscardRule, larastan:PHPStan\Rules\Methods\CallToStaticMethodStatementWithNoDiscardRule |
| Consider | Keep goto targets defined within the same function scope | larastan:PHPStan\Rules\Keywords\GotoUndefinedLabelRule |
| Consider | Attach var annotations to the actual variable they describe | larastan:PHPStan\Rules\PhpDoc\WrongVariableNameInVarTagRule |
| Consider | Use ObservedBy for equivalent observer registration when adopting model attributes | shift:workbench/laravel-observedby-attribute |
| Consider | Replace removed legacy Laravel helpers, classes and method names with supported equivalents | partial: rector:RenameMethodRector, partial: rector:FuncCallToStaticCallRector, partial: rector:RenameClassRector |
| Consider | Call translation query scopes through an explicit model query builder | larastan:NoImplicitQueryBuilderCallRule |
| Consider | Include the guard in manually constructed login events | partial: rector:AddGuardToLoginEventRector |
| Consider | Compare full timestamps when the cutoff includes time | partial: rector:ChangeQueryWhereDateValueWithCarbonRector |
| Consider | Pass view data as explicit key-value arrays | shift:workbench/normalize-view-data |
| Consider | Inject the Blade compiler when adopting dependency injection | rector:StaticCallToMethodCallRector |
| Consider | Read SQL expressions through the current grammar-aware API | rector:DatabaseExpressionToStringToMethodCallRector, rector:DatabaseExpressionCastsToMethodCallRector |
| Consider | Use the portable whereLike API for supported pattern searches | rector:WhereToWhereLikeRector |
| Consider | Use the current Cashier Stripe client accessor | rector:CashierStripeOptionsToStripeRector |
| Consider | Query PHPStan types through the Type API | larastan:PHPStan\Rules\Api\ApiInstanceofTypeRule |
| Consider | Use Collects metadata under the Laravel 13 attribute convention | rector:CollectsPropertyToCollectsAttributeRector |
| Consider | Use the dedicated permanent redirect helper for an intended 301 | rector:Redirect301ToPermanentRedirectRector |
| Consider | Construct JsonResponse directly when adopting that response convention | rector:ResponseHelperCallToJsonResponseRector |
| Consider | Express integer cache lifetimes in seconds when migrating code that used minute-based cache APIs | rector:MinutesToSecondsInCacheRector |
| Consider | Use short [] syntax for array destructuring assignments | pint:list_syntax |
| Consider | Use brace syntax for PHP control structures instead of colon/end syntax | pint:no_alternative_syntax |
| Consider | Omit the closing ?> tag from files containing only PHP | pint:no_closing_tag |
| Consider | Use echo consistently instead of mixing echo and print | pint:no_mixed_echo_print |
| Consider | Use square brackets instead of curly braces for array and string offsets | pint:normalize_index_brace |
| Consider | Expand resource and apiResource registrations into individual routes when adopting explicitly listed routes | shift:workbench/expand-resource-routes |
| Consider | Inject HTTP and session collaborators when adopting explicit dependency injection | partial: rector:ArgumentFuncCallToMethodCallRector, partial: rector:StaticCallToMethodCallRector |
| Consider | Keep response-cacheable static routes in a separate routes file | shift:public/fast-laravel/routes-file |
| Consider | Re-register existing HTTP middleware when migrating to the streamlined application structure | partial: shift:public/slimmer/middleware |
| Consider | Simplify XOR expressions with constant operands | larastan:PHPStan\Rules\Comparison\LogicalXorConstantConditionRule |
| Consider | Preserve schedules when changing application structure | partial: shift:public/slimmer/scheduler |
| Consider | Move legacy Lumen applications onto a maintained Laravel upgrade path | partial: shift:lumen-to-laravel |
| Consider | Use context-independent callable syntax | partial: rector:ArrayToFirstClassCallableRector |
| Consider | Use array unpacking only with equivalent key semantics | shift:workbench/adopt-array-unpacking |
| Consider | Migrate custom skeleton behavior before deleting old files | partial: shift:public/slimmer/core-files |
| Consider | Use equivalent Laravel helpers consistently | rector:DispatchToHelperFunctionsRector, shift:workbench/adopt-common-helpers |
| Consider | Inject dispatcher services when using dependency injection style | partial: rector:ArgumentFuncCallToMethodCallRector, partial: rector:StaticCallToMethodCallRector |
| Consider | Replace the legacy model event-map property | rector:RenamePropertyRector |
| Consider | Resolve infrastructure services by class or contract | rector:ReplaceServiceContainerCallArgRector |
| Consider | Match the current dispatcher listener signature | rector:AddArgumentDefaultValueRector |
| Consider | Use the Laravel request object under persistent workers | partial: rector:RequestVariablesToRequestFacadeRector, partial: rector:ServerVariableToRequestFacadeRector |
| Consider | Preserve generator key and value contracts when delegating | larastan:PHPStan\Rules\Generators\YieldFromTypeRule |
| Consider | Document checked exceptions that escape callable boundaries | larastan:PHPStan\Rules\Exceptions\MissingCheckedExceptionInFunctionThrowsRule, larastan:PHPStan\Rules\Exceptions\MissingCheckedExceptionInMethodThrowsRule, larastan:PHPStan\Rules\Exceptions\MissingCheckedExceptionInPropertyHookThrowsRule |
| Consider | Use supported relationship paths for aggregate methods | partial: larastan:RelationExistenceRule |
| Consider | Serialize queued model references using the framework model serialization traits | partial: larastan:JobWithModelPropertyDeclaresSerializesModelsRule |
| Consider | Combine equivalent consecutive isset and unset operations | pint:combine_consecutive_issets, pint:combine_consecutive_unsets |
| Consider | Brace interpolated variables under the explicit interpolation policy | pint:explicit_string_variable |
| Consider | Resolve only declared template names in PHPStan extensions | larastan:PHPStan\Rules\Api\GetTemplateTypeRule |
| Consider | Fix errors that prevent PHPStan from analyzing the program | partial: larastan:PHPStan\Rules\Classes\ExistingClassInClassExtendsRule |
| Consider | Use long echo tags under the selected Symfony formatter policy | pint:echo_tag_syntax |
| Consider | Use while true for intentional unbounded loops under Symfony style | pint:empty_loop_condition |
| Consider | Combine adjacent constant string literals | pint:no_useless_concat_operator |
| Consider | Terminate PHP instructions before closing template tags | pint:semicolon_after_instruction |
| Consider | Use prefix increments under the selected Symfony policy | pint:increment_style |
| Consider | Use ordinary comments for prose inside method bodies | pint:phpdoc_to_comment |
| Consider | Omit empty constructor parentheses under Laravel style | pint:new_with_parentheses |
| Consider | Lead continuation lines with boolean operators | pint:operator_linebreak |
| Consider | Place null last in native union types | pint:ordered_types |
| Consider | Remove inner attribute bracket whitespace | pint:attribute_block_no_spaces |
| Consider | Place trait imports first in classes | pint:ordered_class_elements |
| Consider | Align multiline doc-comment asterisks | pint:align_multiline_comment |
| Consider | Fully qualify globals under the Symfony policy | pint:global_namespace_import |
| Consider | Remove empty ordinary comments | partial: pint:no_empty_comment |
| Consider | Omit redundant null defaults on untyped properties | pint:no_null_property_initialization |
| Consider | Remove else after a terminating branch | pint:no_useless_else |
| Consider | Use ordinary access on proven non-null receivers | larastan:PHPStan\Rules\Methods\NullsafeMethodCallRule, larastan:PHPStan\Rules\Properties\NullsafePropertyFetchRule, pint:no_useless_nullsafe_operator |
| Consider | Omit terminal periods from annotation descriptions under Symfony style | pint:phpdoc_annotation_without_dot |
| Consider | Place null last in PHPDoc unions under Symfony style | pint:phpdoc_types_order |
| Consider | Use private visibility where inheritance cannot use protected | pint:protected_to_private |
| Consider | Keep throw expressions together under Symfony style | pint:single_line_throw |
| Consider | Use increment syntax for single-step updates under Symfony style | pint:standardize_increment |
| Consider | Use break to exit a switch case | pint:switch_continue_to_break |
| Consider | Space native union pipes under the selected custom style | pint:types_spaces |
| Consider | Order class elements consistently | pint:ordered_class_elements |
| Consider | Keep sealed hierarchies valid | larastan:PHPStan\Rules\Classes\AllowedSubTypesRule, larastan:PHPStan\Rules\PhpDoc\SealedDefinitionTraitRule, larastan:PHPStan\Rules\PhpDoc\SealedDefinitionClassRule |
| Consider | Implement real interfaces on enums | larastan:PHPStan\Rules\Classes\ExistingClassesInEnumImplementsRule |
| Consider | Keep local PHPDoc type aliases resolvable | larastan:PHPStan\Rules\Classes\LocalTypeAliasesRule, larastan:PHPStan\Rules\Classes\LocalTypeTraitAliasesRule, larastan:PHPStan\Rules\Classes\LocalTypeTraitUseAliasesRule |
| Consider | Declare and satisfy required base classes | larastan:PHPStan\Rules\Classes\RequireExtendsRule, larastan:PHPStan\Rules\PhpDoc\RequireExtendsDefinitionClassRule, larastan:PHPStan\Rules\PhpDoc\RequireExtendsDefinitionTraitRule |
| Consider | Use contextual magic constants in meaningful contexts | partial: larastan:PHPStan\Rules\Constants\MagicConstantContextRule |
| Consider | Use valid interval specifications | partial: larastan:PHPStan\Rules\DateIntervalInstantiationRule |
| Consider | Keep no-throws declarations honest | larastan:PHPStan\Rules\Exceptions\ThrowsVoidFunctionWithExplicitThrowPointRule, larastan:PHPStan\Rules\Exceptions\ThrowsVoidMethodWithExplicitThrowPointRule, larastan:PHPStan\Rules\Exceptions\ThrowsVoidPropertyHookWithExplicitThrowPointRule |
| Consider | Do not return nullsafe expressions by reference | partial: larastan:PHPStan\Rules\Functions\ArrowFunctionReturnNullsafeByRefRule, partial: larastan:PHPStan\Rules\Functions\ReturnNullsafeByRefRule |
| Consider | Use string names for dynamic class constants | partial: larastan:PHPStan\Rules\Constants\DynamicClassConstantFetchRule |
| Consider | Respect template variance positions | larastan:PHPStan\Rules\Generics\FunctionSignatureVarianceRule, larastan:PHPStan\Rules\Generics\MethodSignatureVarianceRule, larastan:PHPStan\Rules\Generics\PropertyVarianceRule |
| Consider | Apply callable timing annotations to callable parameters | partial: larastan:PHPStan\Rules\PhpDoc\IncompatibleParamImmediatelyInvokedCallableRule |
| Consider | Pipe values into compatible callables | larastan:PHPStan\Rules\Operators\PipeOperatorRule |
| Consider | Describe receiver state changes with valid self-out types | larastan:PHPStan\Rules\PhpDoc\IncompatibleSelfOutTypeRule, larastan:PHPStan\Rules\Methods\MissingMethodSelfOutTypeRule |
| Consider | Honor PHPDoc readonly property lifecycles | larastan:PHPStan\Rules\Properties\MissingReadOnlyByPhpDocPropertyAssignRule, larastan:PHPStan\Rules\Properties\ReadOnlyByPhpDocPropertyRule, larastan:PHPStan\Rules\Properties\ReadOnlyByPhpDocPropertyAssignRefRule, larastan:PHPStan\Rules\Properties\ReadOnlyByPhpDocPropertyAssignRule |
| Consider | Respect property write visibility for references | larastan:PHPStan\Rules\Properties\PropertyAssignRefRule |
| Consider | Keep set-hook input types compatible | larastan:PHPStan\Rules\Properties\SetPropertyHookParameterRule |
| Consider | Keep purity promises truthful | larastan:PHPStan\Rules\Pure\PureFunctionRule, larastan:PHPStan\Rules\Pure\PureMethodRule, larastan:PHPStan\Rules\Pure\PurePropertyHookRule |
| Consider | Keep parameter output types precise | larastan:PHPStan\Rules\TooWideTypehints\TooWideFunctionParameterOutTypeRule, larastan:PHPStan\Rules\TooWideTypehints\TooWideMethodParameterOutTypeRule |
| Consider | Prefer native readonly over documentation-only promises | shift:workbench/set-readonly-properties |
| Consider | Run the maintained Dusk suite in CI | partial: shift:public/ci/dusk |
| Consider | Preserve element types with PHPDoc generics | partial: larastan:PHPStan\Rules\Generics\ClassAncestorsRule, partial: larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule, partial: rector:AddGenericReturnTypeToRelationsRector, partial: rector:AddExtendsAnnotationToModelFactoriesRector |
| Consider | Depend on the validator contract in consumers | rector:RenameClassRector |
| Consider | Use validateResolved for resolution-triggered validation | rector:RenameMethodRector |
| Consider | Keep Form Request validationData public | rector:ChangeMethodVisibilityRector |

## Config snippets

### larastan:BatchableJobChecksCancellationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkBatchableJobChecksCancellation: true

```

### larastan:BatchedJobIsBatchableRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkBatchedJobIsBatchable: true

```

### larastan:CheckDispatchArgumentTypesCompatibleWithClassConstructorRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8

```

### larastan:ConfigCollectionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkConfigTypes: true

```

### larastan:DeferrableServiceProviderMissingProvidesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8

```

### larastan:JobDispatchedInTransactionUsesAfterCommitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkDispatchInTransactionAfterCommit: true

```

### larastan:JobWithModelPropertyDeclaresSerializesModelsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkJobSerializesModels: true

```

### larastan:ModelAppendsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkModelAppends: true

```

### larastan:ModelRelationDefaultsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8

```

### larastan:NoAuthFacadeInRequestScopeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkAuthCallsWhenInRequestScope: true

```

### larastan:NoAuthHelperInRequestScopeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkAuthCallsWhenInRequestScope: true

```

### larastan:NoBatchedUniqueJobRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    noBatchedUniqueJob: true

```

### larastan:NoEnvCallsOutsideOfConfigRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    noEnvCallsOutsideOfConfig: true

```

### larastan:NoImplicitQueryBuilderCallRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    noImplicitQueryBuilderCall: true

```

### larastan:NoMissingTranslationsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkMissingTranslations: true

```

### larastan:NoModelMakeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    noModelMake: true

```

### larastan:NoPublicModelScopeAndAccessorRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkModelMethodVisibility: true

```

### larastan:NoUnnecessaryCollectionCallRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    noUnnecessaryCollectionCall: true

```

### larastan:NoUnnecessaryEnumerableToArrayCallsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    noUnnecessaryEnumerableToArrayCalls: true

```

### larastan:NoUselessValueFunctionCallsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8

```

### larastan:NoUselessWithFunctionCallsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8

```

### larastan:OctaneCompatibilityRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkOctaneCompatibility: true

```

### larastan:PHPStan\Rules\Api\ApiInstanceofTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Api\GetTemplateTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Api\NodeConnectingVisitorAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Api\OldPhpParser4ClassRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Api\RuntimeReflectionFunctionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Api\RuntimeReflectionInstantiationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\ArrayDestructuringRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\ArrayUnpackingRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\DeadForeachRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\DuplicateKeysInLiteralArraysRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\InvalidKeyInArrayDimFetchRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\InvalidKeyInArrayItemRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\IterableInForeachRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\NonexistentOffsetInArrayDimFetchRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\OffsetAccessAssignOpRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\OffsetAccessAssignmentRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\OffsetAccessValueAssignmentRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\OffsetAccessWithoutDimForReadingRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Arrays\UnpackIterableInArrayRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Cast\EchoRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Cast\InvalidCastRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Cast\InvalidPartOfEncapsedStringRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Cast\PrintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Cast\UnsetCastRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Cast\VoidCastRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\AccessPrivateConstantThroughStaticRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\AllowedSubTypesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ClassAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ClassConstantAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ClassConstantRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\DuplicateClassDeclarationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\DuplicateDeclarationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\DuplicateTraitDeclarationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\EnumSanityRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ExistingClassInClassExtendsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ExistingClassInInstanceOfRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ExistingClassInTraitUseRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ExistingClassesInClassImplementsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ExistingClassesInEnumImplementsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ExistingClassesInInterfaceExtendsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\ImpossibleInstanceOfRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\InstantiationCallableRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\InstantiationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\LocalTypeAliasesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\LocalTypeTraitAliasesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\LocalTypeTraitUseAliasesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\MethodTagRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\MethodTagTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\MethodTagTraitUseRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\MixinRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\MixinTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\MixinTraitUseRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\NewStaticInAbstractClassStaticMethodRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        newStaticInAbstractClassStaticMethod: true

```

### larastan:PHPStan\Rules\Classes\NewStaticRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\NonClassAttributeClassRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\PropertyTagRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\PropertyTagTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\PropertyTagTraitUseRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\RequireExtendsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\RequireImplementsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\TraitAttributeClassRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Classes\UnusedConstructorParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\BooleanAndConstantConditionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\BooleanNotConstantConditionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\ConstantConditionInTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\DoWhileLoopConstantConditionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\ImpossibleCheckTypeFunctionCallRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\ImpossibleCheckTypeMethodCallRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\ImpossibleCheckTypeStaticMethodCallRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\ImpossibleInArrayHaystackFiniteTypesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        finiteTypesInHaystack: true

```

### larastan:PHPStan\Rules\Comparison\LogicalXorConstantConditionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\MatchExpressionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\SwitchConditionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        switchConditionAlwaysFalse: true

```

### larastan:PHPStan\Rules\Comparison\UsageOfVoidMatchExpressionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Comparison\WhileLoopAlwaysTrueConditionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\ClassAsClassConstantRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\ConstantAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\ConstantRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\DynamicClassConstantFetchRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\FinalPrivateConstantRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\MagicConstantContextRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\MissingClassConstantTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\OverridingConstantRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\ValueAssignedToClassConstantRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Constants\ValueAssignedToDefineRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        checkDynamicConstantNameValues: true

```

### larastan:PHPStan\Rules\Constants\ValueAssignedToGlobalConstantRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        checkDynamicConstantNameValues: true

```

### larastan:PHPStan\Rules\DateIntervalInstantiationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        checkDateIntervalConstructor: true

```

### larastan:PHPStan\Rules\DateTimeInstantiationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\CallToConstructorStatementWithoutImpurePointsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\CallToFunctionStatementWithoutImpurePointsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\CallToMethodStatementWithoutImpurePointsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\CallToStaticMethodStatementWithoutImpurePointsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\NoopRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\UnreachableStatementRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\UnusedPrivateConstantRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\UnusedPrivateMethodRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\UnusedPrivatePropertyRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\DeadCode\UnusedVariableRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        unusedVariable: true

```

### larastan:PHPStan\Rules\EnumCases\EnumCaseAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Exceptions\CatchWithUnthrownExceptionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Exceptions\CaughtExceptionExistenceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Exceptions\MethodThrowTypeCovarianceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    exceptions:
        check:
            throwTypeCovariance: true

```

### larastan:PHPStan\Rules\Exceptions\MissingCheckedExceptionInFunctionThrowsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    exceptions:
        check:
            missingCheckedExceptionInThrows: true

```

### larastan:PHPStan\Rules\Exceptions\MissingCheckedExceptionInMethodThrowsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    exceptions:
        check:
            missingCheckedExceptionInThrows: true

```

### larastan:PHPStan\Rules\Exceptions\MissingCheckedExceptionInPropertyHookThrowsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    exceptions:
        check:
            missingCheckedExceptionInThrows: true

```

### larastan:PHPStan\Rules\Exceptions\OverwrittenExitPointByFinallyRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Exceptions\ThrowExprTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Exceptions\ThrowsVoidFunctionWithExplicitThrowPointRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Exceptions\ThrowsVoidMethodWithExplicitThrowPointRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Exceptions\ThrowsVoidPropertyHookWithExplicitThrowPointRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Exceptions\TooWideFunctionThrowTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    exceptions:
        check:
            tooWideThrowType: true

```

### larastan:PHPStan\Rules\Exceptions\TooWideMethodThrowTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    exceptions:
        check:
            tooWideThrowType: true

```

### larastan:PHPStan\Rules\Exceptions\TooWidePropertyHookThrowTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    exceptions:
        check:
            tooWideThrowType: true

```

### larastan:PHPStan\Rules\Functions\ArrayFilterRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ArrayValuesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ArrowFunctionAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ArrowFunctionReturnNullsafeByRefRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ArrowFunctionReturnTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\CallCallablesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\CallToFunctionParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    checkModelProperties: true

```

### larastan:PHPStan\Rules\Functions\CallToFunctionStatementWithNoDiscardRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\CallToFunctionStatementWithoutSideEffectsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\CallToNonExistentFunctionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ClosureAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ClosureReturnTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\DefineParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\DuplicateFunctionDeclarationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ExistingClassesInArrowFunctionTypehintsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ExistingClassesInClosureTypehintsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ExistingClassesInTypehintsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\FilterVarRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\FunctionAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\FunctionCallableRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ImplodeParameterCastableToStringRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\IncompatibleArrowFunctionDefaultParameterTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\IncompatibleClosureDefaultParameterTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\IncompatibleDefaultParameterTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\InnerFunctionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\InvalidLexicalVariablesInClosureUseRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\InvalidParameterNameRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\MissingFunctionParameterTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\MissingFunctionReturnTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ParameterCastableToNumberRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        checkParameterCastableToNumberFunctions: true

```

### larastan:PHPStan\Rules\Functions\ParameterCastableToStringRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\PrintfArrayParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\PrintfParameterTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        checkPrintfParameterTypes: true

```

### larastan:PHPStan\Rules\Functions\PrintfParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\RandomIntParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\RedefinedParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ReturnNullsafeByRefRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ReturnTypeAfterFinallyRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\ReturnTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\SortWithoutEffectRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        sortWithoutEffect: true

```

### larastan:PHPStan\Rules\Functions\UnusedClosureUsesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Functions\UnusedFunctionParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        unusedParameters: true

```

### larastan:PHPStan\Rules\Functions\UselessFunctionReturnValueRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generators\YieldFromTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generators\YieldInGeneratorRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generators\YieldTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\ClassAncestorsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\ClassTemplateTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\EnumAncestorsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\EnumTemplateTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\FunctionSignatureVarianceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\FunctionTemplateTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\InterfaceAncestorsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\InterfaceTemplateTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\MethodSignatureVarianceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\MethodTagTemplateTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\MethodTagTemplateTypeTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\MethodTemplateTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\PropertyVarianceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Generics\UsedTraitsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Ignore\IgnoreParseErrorRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\InternalTag\RestrictedInternalClassConstantUsageExtension
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        internalTag: true

```

### larastan:PHPStan\Rules\InternalTag\RestrictedInternalClassNameUsageExtension
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        internalTag: true

```

### larastan:PHPStan\Rules\InternalTag\RestrictedInternalFunctionUsageExtension
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        internalTag: true

```

### larastan:PHPStan\Rules\InternalTag\RestrictedInternalMethodUsageExtension
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        internalTag: true

```

### larastan:PHPStan\Rules\InternalTag\RestrictedInternalPropertyUsageExtension
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        internalTag: true

```

### larastan:PHPStan\Rules\Keywords\GotoUndefinedLabelRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Keywords\RequireFileExistsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Keywords\UnusedLabelRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        unusedLabel: true

```

### larastan:PHPStan\Rules\Methods\CallMethodsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    checkModelProperties: true

```

### larastan:PHPStan\Rules\Methods\CallPrivateMethodThroughStaticRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\CallStaticMethodsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    checkModelProperties: true

```

### larastan:PHPStan\Rules\Methods\CallToMethodStatementWithNoDiscardRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\CallToStaticMethodStatementWithNoDiscardRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\ConsistentConstructorDeclarationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\ConsistentConstructorRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\ConstructorReturnTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\ExistingClassesInTypehintsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\FinalPrivateMethodRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\IncompatibleDefaultParameterTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\MethodCallWithPossiblyRenamedNamedArgumentRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\MethodCallableRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\MethodSignatureRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    reportMaybesInMethodSignatures: true

```

### larastan:PHPStan\Rules\Methods\MethodVisibilityInInterfaceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\MissingMagicSerializationMethodsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\MissingMethodImplementationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\MissingMethodParameterTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\MissingMethodReturnTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\MissingMethodSelfOutTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\NullsafeMethodCallRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\OverridingMethodRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    checkMissingOverrideMethodAttribute: true

```

### larastan:PHPStan\Rules\Methods\ReturnTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\StaticMethodCallableRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Methods\UnusedMethodParametersRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        unusedParameters: true

```

### larastan:PHPStan\Rules\Missing\MissingReturnRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Names\UsedNamesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Namespaces\ExistingNamesInGroupUseRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Namespaces\ExistingNamesInUseRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Operators\BacktickRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Operators\InvalidBinaryOperationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Operators\InvalidComparisonOperationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Operators\InvalidIncDecOperationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Operators\InvalidUnaryOperationRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Operators\PipeOperatorRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\FunctionAssertRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\FunctionConditionalReturnTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\IncompatibleClassConstantPhpDocTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\IncompatibleParamImmediatelyInvokedCallableRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\IncompatiblePhpDocTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\IncompatiblePropertyHookPhpDocTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\IncompatiblePropertyPhpDocTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\IncompatibleSelfOutTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\InvalidPHPStanDocTagRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\InvalidPhpDocTagValueRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\InvalidPhpDocVarTagTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\InvalidThrowsPhpDocValueRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\MethodAssertRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\MethodConditionalReturnTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\RequireExtendsDefinitionClassRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\RequireExtendsDefinitionTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\RequireImplementsDefinitionClassRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\RequireImplementsDefinitionTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\SealedDefinitionClassRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\SealedDefinitionTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\VarTagChangedExpressionTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\PhpDoc\WrongVariableNameInVarTagRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\AccessPrivatePropertyThroughStaticRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\AccessPropertiesInAssignRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\AccessPropertiesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\AccessStaticPropertiesInAssignRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\AccessStaticPropertiesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\DefaultValueTypesAssignedToPropertiesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ExistingClassesInPropertiesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ExistingClassesInPropertyHookTypehintsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\GetNonVirtualPropertyHookReadRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\InvalidCallablePropertyTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\MissingPropertyTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\MissingReadOnlyByPhpDocPropertyAssignRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\MissingReadOnlyPropertyAssignRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\NullsafePropertyFetchRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\OverridingPropertyRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\PropertiesInInterfaceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\PropertyAssignRefRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\PropertyInClassRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ReadOnlyByPhpDocPropertyAssignRefRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ReadOnlyByPhpDocPropertyAssignRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ReadOnlyByPhpDocPropertyRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ReadOnlyPropertyAssignRefRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ReadOnlyPropertyAssignRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ReadOnlyPropertyRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\ReadingWriteOnlyPropertiesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\SetNonVirtualPropertyHookAssignRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\SetPropertyHookParameterRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\TypesAssignedToPropertiesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Properties\UninitializedPropertyRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    checkUninitializedProperties: true

```

### larastan:PHPStan\Rules\Properties\WritingToReadOnlyPropertiesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Pure\PureFunctionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Pure\PureMethodRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Pure\PurePropertyHookRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Regexp\RegularExpressionPatternRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Regexp\RegularExpressionQuotingRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\TooWideTypehints\TooWideArrowFunctionReturnTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\TooWideTypehints\TooWideClosureReturnTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\TooWideTypehints\TooWideFunctionParameterOutTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\TooWideTypehints\TooWideFunctionReturnTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\TooWideTypehints\TooWideMethodParameterOutTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\TooWideTypehints\TooWideMethodReturnTypehintRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\TooWideTypehints\TooWidePropertyTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Traits\ConflictingTraitConstantsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Traits\NotAnalysedTraitRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Traits\TraitAttributesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\AssignToByRefExprFromForeachRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max
    featureToggles:
        assignToByRefForeachExpr: true

```

### larastan:PHPStan\Rules\Variables\CompactVariablesRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\DefinedVariableRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\EmptyRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\IssetRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\NullCoalesceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\ParameterOutAssignedTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\ParameterOutExecutionEndTypeRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\UnsetRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Variables\VariableCloningRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:PHPStan\Rules\Whitespace\FileWhitespaceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: max

```

### larastan:RelationExistenceRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8

```

### larastan:UndefinedArgumentOrOptionRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
services:
    -
        class: Larastan\Larastan\Rules\ConsoleCommand\UndefinedArgumentOrOptionRule
        tags: [phpstan.rules.rule]

```

### larastan:UniqueJobDeclaresUniqueForRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkUniqueJobUniqueFor: true

```

### larastan:UniqueJobDeclaresUniqueIdRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkUniqueJobUniqueId: true

```

### larastan:UnusedViewsRule
```
includes:
    - vendor/larastan/larastan/extension.neon
parameters:
    paths:
        - app
    level: 8
    checkUnusedViews: true
# Review dynamic view names before deleting reported views.

```

### pint:Pint/laravel_blade
```
{
  "preset": "laravel",
  "rules": {
    "Pint/laravel_blade": true
  }
}

```

### pint:align_multiline_comment
```
{
  "preset": "symfony",
  "rules": {
    "align_multiline_comment": true
  }
}

```

### pint:array_indentation
```
{
  "preset": "laravel",
  "rules": {
    "array_indentation": true
  }
}

```

### pint:array_syntax
```
{
  "preset": "laravel",
  "rules": {
    "array_syntax": {
      "syntax": "short"
    }
  }
}

```

### pint:attribute_block_no_spaces
```
{
  "preset": "per",
  "rules": {
    "attribute_block_no_spaces": true
  }
}

```

### pint:backtick_to_shell_exec
```
{
  "preset": "symfony",
  "rules": {
    "backtick_to_shell_exec": true
  }
}

```

### pint:binary_operator_spaces
```
{
  "preset": "laravel",
  "rules": {
    "binary_operator_spaces": {
      "default": "single_space"
    }
  }
}

```

### pint:blank_line_after_namespace
```
{
  "preset": "laravel",
  "rules": {
    "blank_line_after_namespace": true
  }
}

```

### pint:blank_line_after_opening_tag
```
{
  "preset": "laravel",
  "rules": {
    "blank_line_after_opening_tag": true
  }
}

```

### pint:blank_line_before_statement
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "blank_line_before_statement": {
      "statements": [
        "continue",
        "return"
      ]
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "blank_line_before_statement": {
      "statements": [
        "break",
        "case",
        "continue",
        "declare",
        "default",
        "do",
        "exit",
        "for",
        "foreach",
        "goto",
        "if",
        "include",
        "include_once",
        "phpdoc",
        "require",
        "require_once",
        "return",
        "switch",
        "throw",
        "try",
        "while",
        "yield",
        "yield_from"
      ]
    }
  }
}
```

Alternative 3:
```json
{
  "preset": "laravel",
  "rules": {
    "blank_line_before_statement": {
      "statements": [
        "break",
        "continue",
        "declare",
        "if",
        "return",
        "throw",
        "try"
      ]
    }
  }
}
```

Alternative 4:
```json
{
  "preset": "laravel",
  "rules": {
    "blank_line_before_statement": {
      "statements": [
        "break",
        "continue",
        "declare",
        "return",
        "throw",
        "try",
        "if",
        "switch",
        "for",
        "foreach",
        "while"
      ]
    }
  }
}
```

```

### pint:blank_line_between_import_groups
```
{
  "preset": "laravel",
  "rules": {
    "blank_line_between_import_groups": true
  }
}

```

### pint:blank_lines_before_namespace
```
{
  "preset": "laravel",
  "rules": {
    "blank_lines_before_namespace": true
  }
}

```

### pint:braces_position
```
{
  "preset": "laravel",
  "rules": {
    "braces_position": {
      "control_structures_opening_brace": "same_line",
      "functions_opening_brace": "next_line_unless_newline_at_signature_end",
      "anonymous_functions_opening_brace": "same_line",
      "classes_opening_brace": "next_line_unless_newline_at_signature_end",
      "anonymous_classes_opening_brace": "next_line_unless_newline_at_signature_end",
      "allow_single_line_empty_anonymous_classes": false,
      "allow_single_line_anonymous_functions": false
    }
  }
}

```

### pint:cast_spaces
```
{
  "preset": "laravel",
  "rules": {
    "cast_spaces": true
  }
}

```

### pint:class_attributes_separation
```
{
  "preset": "laravel",
  "rules": {
    "class_attributes_separation": {
      "elements": {
        "const": "one",
        "method": "one",
        "property": "one",
        "trait_import": "none"
      }
    }
  }
}

```

### pint:class_definition
```
{
  "preset": "laravel",
  "rules": {
    "class_definition": {
      "multi_line_extends_each_single_line": true,
      "single_item_single_line": true,
      "single_line": true
    }
  }
}

```

### pint:class_reference_name_casing
```
{
  "preset": "symfony",
  "rules": {
    "class_reference_name_casing": true
  }
}

```

### pint:clean_namespace
```
{
  "preset": "laravel",
  "rules": {
    "clean_namespace": true
  }
}

```

### pint:combine_consecutive_issets
```
{
  "preset": "laravel",
  "rules": {
    "combine_consecutive_issets": true
  }
}

```

### pint:combine_consecutive_unsets
```
{
  "preset": "laravel",
  "rules": {
    "combine_consecutive_unsets": true
  }
}

```

### pint:compact_nullable_type_declaration
```
{
  "preset": "laravel",
  "rules": {
    "compact_nullable_type_declaration": true
  }
}

```

### pint:concat_space
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "concat_space": {
      "spacing": "none"
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "concat_space": {
      "spacing": "one"
    }
  }
}
```

```

### pint:constant_case
```
{
  "preset": "laravel",
  "rules": {
    "constant_case": {
      "case": "lower"
    }
  }
}

```

### pint:control_structure_braces
```
{
  "preset": "laravel",
  "rules": {
    "control_structure_braces": true
  }
}

```

### pint:control_structure_continuation_position
```
{
  "preset": "laravel",
  "rules": {
    "control_structure_continuation_position": {
      "position": "same_line"
    }
  }
}

```

### pint:declare_equal_normalize
```
{
  "preset": "laravel",
  "rules": {
    "declare_equal_normalize": true
  }
}

```

### pint:declare_parentheses
```
{
  "preset": "laravel",
  "rules": {
    "declare_parentheses": true
  }
}

```

### pint:declare_strict_types
```
{
  "preset": "laravel",
  "rules": {
    "declare_strict_types": true
  }
}

```

### pint:echo_tag_syntax
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "symfony",
  "rules": {
    "echo_tag_syntax": true
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "echo_tag_syntax": {
      "format": "short"
    }
  }
}
```

Alternative 3:
```json
{
  "preset": "laravel",
  "rules": {
    "echo_tag_syntax": {
      "format": "long"
    }
  }
}
```

```

### pint:elseif
```
{
  "preset": "laravel",
  "rules": {
    "elseif": true
  }
}

```

### pint:empty_loop_body
```
{
  "preset": "symfony",
  "rules": {
    "empty_loop_body": {
      "style": "braces"
    }
  }
}

```

### pint:empty_loop_condition
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "symfony",
  "rules": {
    "empty_loop_condition": true
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "empty_loop_condition": {
      "style": "while"
    }
  }
}
```

Alternative 3:
```json
{
  "preset": "laravel",
  "rules": {
    "empty_loop_condition": {
      "style": "for"
    }
  }
}
```

```

### pint:encoding
```
{
  "preset": "laravel",
  "rules": {
    "encoding": true
  }
}

```

### pint:explicit_string_variable
```
{
  "preset": "laravel",
  "rules": {
    "explicit_string_variable": true
  }
}

```

### pint:full_opening_tag
```
{
  "preset": "laravel",
  "rules": {
    "full_opening_tag": true
  }
}

```

### pint:fully_qualified_strict_types
```
{
  "preset": "laravel",
  "rules": {
    "fully_qualified_strict_types": {
      "import_symbols": true,
      "phpdoc_tags": [
        "param",
        "phpstan-param",
        "phpstan-property",
        "phpstan-property-read",
        "phpstan-property-write",
        "phpstan-return",
        "phpstan-var",
        "property",
        "property-read",
        "property-write",
        "psalm-param",
        "psalm-property",
        "psalm-property-read",
        "psalm-property-write",
        "psalm-return",
        "psalm-var",
        "return",
        "see",
        "throws",
        "var",
        "extends",
        "implements",
        "use",
        "phpstan-extends",
        "phpstan-implements",
        "phpstan-use",
        "psalm-extends",
        "psalm-implements",
        "psalm-use",
        "mixin",
        "param-out",
        "param-closure-this",
        "phpstan-param-out",
        "phpstan-assert",
        "phpstan-assert-if-true",
        "phpstan-assert-if-false",
        "phpstan-self-out",
        "phpstan-this-out",
        "phpstan-require-extends",
        "phpstan-require-implements"
      ]
    }
  }
}

```

### pint:function_declaration
```
{
  "preset": "laravel",
  "rules": {
    "function_declaration": true
  }
}

```

### pint:general_phpdoc_tag_rename
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "symfony",
  "rules": {
    "general_phpdoc_tag_rename": {
      "replacements": {
        "inheritDocs": "inheritDoc"
      }
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "general_phpdoc_tag_rename": true
  }
}
```

```

### pint:global_namespace_import
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "symfony",
  "rules": {
    "global_namespace_import": {
      "import_classes": false,
      "import_constants": false,
      "import_functions": false
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "global_namespace_import": {
      "import_classes": true,
      "import_constants": true
    }
  }
}
```

Alternative 3:
```json
{
  "preset": "laravel",
  "rules": {
    "global_namespace_import": {
      "import_classes": true
    }
  }
}
```

Alternative 4:
```json
{
  "preset": "laravel",
  "rules": {
    "global_namespace_import": {
      "import_classes": true,
      "import_functions": true,
      "import_constants": true
    }
  }
}
```

```

### pint:heredoc_to_nowdoc
```
{
  "preset": "laravel",
  "rules": {
    "heredoc_to_nowdoc": true
  }
}

```

### pint:include
```
{
  "preset": "laravel",
  "rules": {
    "include": true
  }
}

```

### pint:increment_style
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "increment_style": {
      "style": "post"
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "increment_style": {
      "style": "pre"
    }
  }
}
```

```

### pint:indentation_type
```
{
  "preset": "laravel",
  "rules": {
    "indentation_type": true
  }
}

```

### pint:integer_literal_case
```
{
  "preset": "laravel",
  "rules": {
    "integer_literal_case": true
  }
}

```

### pint:lambda_not_used_import
```
{
  "preset": "laravel",
  "rules": {
    "lambda_not_used_import": true
  }
}

```

### pint:line_ending
```
{
  "preset": "laravel",
  "rules": {
    "line_ending": true
  }
}

```

### pint:linebreak_after_opening_tag
```
{
  "preset": "laravel",
  "rules": {
    "linebreak_after_opening_tag": true
  }
}

```

### pint:list_syntax
```
{
  "preset": "laravel",
  "rules": {
    "list_syntax": true
  }
}

```

### pint:lowercase_cast
```
{
  "preset": "laravel",
  "rules": {
    "lowercase_cast": true
  }
}

```

### pint:lowercase_keywords
```
{
  "preset": "laravel",
  "rules": {
    "lowercase_keywords": true
  }
}

```

### pint:lowercase_static_reference
```
{
  "preset": "laravel",
  "rules": {
    "lowercase_static_reference": true
  }
}

```

### pint:magic_constant_casing
```
{
  "preset": "laravel",
  "rules": {
    "magic_constant_casing": true
  }
}

```

### pint:magic_method_casing
```
{
  "preset": "laravel",
  "rules": {
    "magic_method_casing": true
  }
}

```

### pint:method_argument_space
```
{
  "preset": "laravel",
  "rules": {
    "method_argument_space": {
      "on_multiline": "ignore"
    }
  }
}

```

### pint:method_chaining_indentation
```
{
  "preset": "laravel",
  "rules": {
    "method_chaining_indentation": true
  }
}

```

### pint:modifier_keywords
```
{
  "preset": "laravel",
  "rules": {
    "modifier_keywords": {
      "elements": [
        "method",
        "property"
      ]
    }
  }
}

```

### pint:multiline_whitespace_before_semicolons
```
{
  "preset": "laravel",
  "rules": {
    "multiline_whitespace_before_semicolons": {
      "strategy": "no_multi_line"
    }
  }
}

```

### pint:native_function_casing
```
{
  "preset": "laravel",
  "rules": {
    "native_function_casing": true
  }
}

```

### pint:native_type_declaration_casing
```
{
  "preset": "laravel",
  "rules": {
    "native_type_declaration_casing": true
  }
}

```

### pint:new_with_parentheses
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "new_with_parentheses": {
      "named_class": false,
      "anonymous_class": false
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "new_with_parentheses": {
      "named_class": true,
      "anonymous_class": true
    }
  }
}
```

```

### pint:no_alias_functions
```
{
  "preset": "laravel",
  "rules": {
    "no_alias_functions": true
  }
}

```

### pint:no_alias_language_construct_call
```
{
  "preset": "laravel",
  "rules": {
    "no_alias_language_construct_call": true
  }
}

```

### pint:no_alternative_syntax
```
{
  "preset": "laravel",
  "rules": {
    "no_alternative_syntax": true
  }
}

```

### pint:no_binary_string
```
{
  "preset": "laravel",
  "rules": {
    "no_binary_string": true
  }
}

```

### pint:no_blank_lines_after_class_opening
```
{
  "preset": "laravel",
  "rules": {
    "no_blank_lines_after_class_opening": true
  }
}

```

### pint:no_blank_lines_after_phpdoc
```
{
  "preset": "laravel",
  "rules": {
    "no_blank_lines_after_phpdoc": true
  }
}

```

### pint:no_break_comment
```
{
  "preset": "per",
  "rules": {
    "no_break_comment": true
  }
}

```

### pint:no_closing_tag
```
{
  "preset": "laravel",
  "rules": {
    "no_closing_tag": true
  }
}

```

### pint:no_empty_comment
```
{
  "preset": "symfony",
  "rules": {
    "no_empty_comment": true
  }
}

```

### pint:no_empty_phpdoc
```
{
  "preset": "laravel",
  "rules": {
    "no_empty_phpdoc": true
  }
}

```

### pint:no_empty_statement
```
{
  "preset": "laravel",
  "rules": {
    "no_empty_statement": true
  }
}

```

### pint:no_extra_blank_lines
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "no_extra_blank_lines": {
      "tokens": [
        "extra",
        "throw",
        "use"
      ]
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "no_extra_blank_lines": {
      "tokens": [
        "curly_brace_block",
        "extra",
        "throw",
        "use"
      ]
    }
  }
}
```

```

### pint:no_leading_import_slash
```
{
  "preset": "laravel",
  "rules": {
    "no_leading_import_slash": true
  }
}

```

### pint:no_leading_namespace_whitespace
```
{
  "preset": "laravel",
  "rules": {
    "no_leading_namespace_whitespace": true
  }
}

```

### pint:no_mixed_echo_print
```
{
  "preset": "laravel",
  "rules": {
    "no_mixed_echo_print": {
      "use": "echo"
    }
  }
}

```

### pint:no_multiline_whitespace_around_double_arrow
```
{
  "preset": "laravel",
  "rules": {
    "no_multiline_whitespace_around_double_arrow": true
  }
}

```

### pint:no_multiple_statements_per_line
```
{
  "preset": "laravel",
  "rules": {
    "no_multiple_statements_per_line": true
  }
}

```

### pint:no_null_property_initialization
```
{
  "preset": "symfony",
  "rules": {
    "no_null_property_initialization": true
  }
}

```

### pint:no_short_bool_cast
```
{
  "preset": "laravel",
  "rules": {
    "no_short_bool_cast": true
  }
}

```

### pint:no_singleline_whitespace_before_semicolons
```
{
  "preset": "laravel",
  "rules": {
    "no_singleline_whitespace_before_semicolons": true
  }
}

```

### pint:no_space_around_double_colon
```
{
  "preset": "laravel",
  "rules": {
    "no_space_around_double_colon": true
  }
}

```

### pint:no_spaces_after_function_name
```
{
  "preset": "laravel",
  "rules": {
    "no_spaces_after_function_name": true
  }
}

```

### pint:no_spaces_around_offset
```
{
  "preset": "laravel",
  "rules": {
    "no_spaces_around_offset": {
      "positions": [
        "inside",
        "outside"
      ]
    }
  }
}

```

### pint:no_superfluous_phpdoc_tags
```
{
  "preset": "laravel",
  "rules": {
    "no_superfluous_phpdoc_tags": {
      "allow_mixed": true,
      "allow_unused_params": true
    }
  }
}

```

### pint:no_trailing_comma_in_singleline
```
{
  "preset": "laravel",
  "rules": {
    "no_trailing_comma_in_singleline": true
  }
}

```

### pint:no_trailing_whitespace
```
{
  "preset": "laravel",
  "rules": {
    "no_trailing_whitespace": true
  }
}

```

### pint:no_trailing_whitespace_in_comment
```
{
  "preset": "laravel",
  "rules": {
    "no_trailing_whitespace_in_comment": true
  }
}

```

### pint:no_unneeded_braces
```
{
  "preset": "laravel",
  "rules": {
    "no_unneeded_braces": true
  }
}

```

### pint:no_unneeded_control_parentheses
```
{
  "preset": "laravel",
  "rules": {
    "no_unneeded_control_parentheses": {
      "statements": [
        "break",
        "clone",
        "continue",
        "echo_print",
        "return",
        "switch_case",
        "yield"
      ]
    }
  }
}

```

### pint:no_unneeded_import_alias
```
{
  "preset": "laravel",
  "rules": {
    "no_unneeded_import_alias": true
  }
}

```

### pint:no_unreachable_default_argument_value
```
{
  "preset": "laravel",
  "rules": {
    "no_unreachable_default_argument_value": true
  }
}

```

### pint:no_unset_cast
```
{
  "preset": "laravel",
  "rules": {
    "no_unset_cast": true
  }
}

```

### pint:no_unused_imports
```
{
  "preset": "laravel",
  "rules": {
    "no_unused_imports": true
  }
}

```

### pint:no_useless_concat_operator
```
{
  "preset": "symfony",
  "rules": {
    "no_useless_concat_operator": true
  }
}

```

### pint:no_useless_else
```
{
  "preset": "symfony",
  "rules": {
    "no_useless_else": true
  }
}

```

### pint:no_useless_nullsafe_operator
```
{
  "preset": "symfony",
  "rules": {
    "no_useless_nullsafe_operator": true
  }
}

```

### pint:no_useless_return
```
{
  "preset": "laravel",
  "rules": {
    "no_useless_return": true
  }
}

```

### pint:no_whitespace_before_comma_in_array
```
{
  "preset": "laravel",
  "rules": {
    "no_whitespace_before_comma_in_array": true
  }
}

```

### pint:no_whitespace_in_blank_line
```
{
  "preset": "laravel",
  "rules": {
    "no_whitespace_in_blank_line": true
  }
}

```

### pint:normalize_index_brace
```
{
  "preset": "laravel",
  "rules": {
    "normalize_index_brace": true
  }
}

```

### pint:not_operator_with_successor_space
```
{
  "preset": "laravel",
  "rules": {
    "not_operator_with_successor_space": true
  }
}

```

### pint:nullable_type_declaration
```
{
  "preset": "laravel",
  "rules": {
    "nullable_type_declaration": true
  }
}

```

### pint:nullable_type_declaration_for_default_null_value
```
{
  "preset": "laravel",
  "rules": {
    "nullable_type_declaration_for_default_null_value": true
  }
}

```

### pint:numeric_literal_separator
```
{
  "preset": "laravel",
  "rules": {
    "numeric_literal_separator": {
      "strategy": "use_separator"
    }
  }
}

```

### pint:object_operator_without_whitespace
```
{
  "preset": "laravel",
  "rules": {
    "object_operator_without_whitespace": true
  }
}

```

### pint:operator_linebreak
```
{
  "preset": "per",
  "rules": {
    "operator_linebreak": true
  }
}

```

### pint:ordered_class_elements
```
{
  "preset": "per",
  "rules": {
    "ordered_class_elements": {
      "order": [
        "use_trait"
      ]
    }
  }
}

```

### pint:ordered_imports
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "ordered_imports": {
      "sort_algorithm": "alpha",
      "imports_order": [
        "const",
        "class",
        "function"
      ]
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "ordered_imports": {
      "imports_order": [
        "class",
        "function",
        "const"
      ],
      "sort_algorithm": "alpha"
    }
  }
}
```

```

### pint:ordered_interfaces
```
{
  "preset": "laravel",
  "rules": {
    "ordered_interfaces": true
  }
}

```

### pint:ordered_traits
```
{
  "preset": "laravel",
  "rules": {
    "ordered_traits": true
  }
}

```

### pint:ordered_types
```
{
  "preset": "per",
  "rules": {
    "ordered_types": {
      "null_adjustment": "always_last",
      "sort_algorithm": "none"
    }
  }
}

```

### pint:php_unit_method_casing
```
{
  "preset": "laravel",
  "rules": {
    "php_unit_method_casing": {
      "case": "snake_case"
    }
  }
}

```

### pint:php_unit_set_up_tear_down_visibility
```
{
  "preset": "laravel",
  "rules": {
    "php_unit_set_up_tear_down_visibility": true
  }
}

```

### pint:phpdoc_align
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_align": {
      "align": "left",
      "spacing": {
        "param": 2
      }
    }
  }
}

```

### pint:phpdoc_annotation_without_dot
```
{
  "preset": "symfony",
  "rules": {
    "phpdoc_annotation_without_dot": true
  }
}

```

### pint:phpdoc_indent
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_indent": true
  }
}

```

### pint:phpdoc_inline_tag_normalizer
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_inline_tag_normalizer": true
  }
}

```

### pint:phpdoc_no_access
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_no_access": true
  }
}

```

### pint:phpdoc_no_alias_tag
```
{
  "preset": "symfony",
  "rules": {
    "phpdoc_no_alias_tag": {
      "replacements": {
        "const": "var",
        "link": "see",
        "property-read": "property",
        "property-write": "property",
        "type": "var"
      }
    }
  }
}

```

### pint:phpdoc_no_package
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_no_package": true
  }
}

```

### pint:phpdoc_no_useless_inheritdoc
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_no_useless_inheritdoc": true
  }
}

```

### pint:phpdoc_order
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_order": {
      "order": [
        "param",
        "return",
        "throws"
      ]
    }
  }
}

```

### pint:phpdoc_return_self_reference
```
{
  "preset": "symfony",
  "rules": {
    "phpdoc_return_self_reference": true
  }
}

```

### pint:phpdoc_scalar
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_scalar": true
  }
}

```

### pint:phpdoc_separation
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_separation": {
      "groups": [
        [
          "deprecated",
          "link",
          "see",
          "since"
        ],
        [
          "author",
          "copyright",
          "license"
        ],
        [
          "category",
          "package",
          "subpackage"
        ],
        [
          "property",
          "property-read",
          "property-write"
        ],
        [
          "param",
          "return"
        ]
      ]
    }
  }
}

```

### pint:phpdoc_single_line_var_spacing
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_single_line_var_spacing": true
  }
}

```

### pint:phpdoc_summary
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_summary": true
  }
}

```

### pint:phpdoc_tag_type
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_tag_type": {
      "tags": {
        "inheritdoc": "inline"
      }
    }
  }
}

```

### pint:phpdoc_to_comment
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "phpdoc_to_comment": true
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "phpdoc_to_comment": {
      "ignored_tags": [
        "var",
        "phpstan-var",
        "psalm-var"
      ]
    }
  }
}
```

```

### pint:phpdoc_trim
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_trim": true
  }
}

```

### pint:phpdoc_trim_consecutive_blank_line_separation
```
{
  "preset": "symfony",
  "rules": {
    "phpdoc_trim_consecutive_blank_line_separation": true
  }
}

```

### pint:phpdoc_types
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_types": true
  }
}

```

### pint:phpdoc_types_order
```
{
  "preset": "symfony",
  "rules": {
    "phpdoc_types_order": {
      "null_adjustment": "always_last",
      "sort_algorithm": "none"
    }
  }
}

```

### pint:phpdoc_var_annotation_correct_order
```
{
  "preset": "symfony",
  "rules": {
    "phpdoc_var_annotation_correct_order": true
  }
}

```

### pint:phpdoc_var_without_name
```
{
  "preset": "laravel",
  "rules": {
    "phpdoc_var_without_name": true
  }
}

```

### pint:protected_to_private
```
{
  "preset": "symfony",
  "rules": {
    "protected_to_private": true
  }
}

```

### pint:psr_autoloading
```
{
  "preset": "laravel",
  "rules": {
    "psr_autoloading": true
  }
}

```

### pint:return_type_declaration
```
{
  "preset": "laravel",
  "rules": {
    "return_type_declaration": {
      "space_before": "none"
    }
  }
}

```

### pint:self_static_accessor
```
{
  "preset": "laravel",
  "rules": {
    "self_static_accessor": true
  }
}

```

### pint:semicolon_after_instruction
```
{
  "preset": "symfony",
  "rules": {
    "semicolon_after_instruction": true
  }
}

```

### pint:short_scalar_cast
```
{
  "preset": "laravel",
  "rules": {
    "short_scalar_cast": true
  }
}

```

### pint:simple_to_complex_string_variable
```
{
  "preset": "symfony",
  "rules": {
    "simple_to_complex_string_variable": true
  }
}

```

### pint:single_blank_line_at_eof
```
{
  "preset": "laravel",
  "rules": {
    "single_blank_line_at_eof": true
  }
}

```

### pint:single_class_element_per_statement
```
{
  "preset": "laravel",
  "rules": {
    "single_class_element_per_statement": {
      "elements": [
        "const",
        "property"
      ]
    }
  }
}

```

### pint:single_import_per_statement
```
{
  "preset": "laravel",
  "rules": {
    "single_import_per_statement": true
  }
}

```

### pint:single_line_after_imports
```
{
  "preset": "laravel",
  "rules": {
    "single_line_after_imports": true
  }
}

```

### pint:single_line_comment_spacing
```
{
  "preset": "laravel",
  "rules": {
    "single_line_comment_spacing": true
  }
}

```

### pint:single_line_comment_style
```
{
  "preset": "laravel",
  "rules": {
    "single_line_comment_style": {
      "comment_types": [
        "hash"
      ]
    }
  }
}

```

### pint:single_line_empty_body
```
{
  "preset": "laravel",
  "rules": {
    "single_line_empty_body": true
  }
}

```

### pint:single_line_throw
```
{
  "preset": "symfony",
  "rules": {
    "single_line_throw": true
  }
}

```

### pint:single_quote
```
{
  "preset": "laravel",
  "rules": {
    "single_quote": true
  }
}

```

### pint:single_space_around_construct
```
{
  "preset": "laravel",
  "rules": {
    "single_space_around_construct": true
  }
}

```

### pint:single_trait_insert_per_statement
```
{
  "preset": "per",
  "rules": {
    "single_trait_insert_per_statement": true
  }
}

```

### pint:space_after_semicolon
```
{
  "preset": "laravel",
  "rules": {
    "space_after_semicolon": true
  }
}

```

### pint:spaces_inside_parentheses
```
{
  "preset": "laravel",
  "rules": {
    "spaces_inside_parentheses": true
  }
}

```

### pint:standardize_increment
```
{
  "preset": "symfony",
  "rules": {
    "standardize_increment": true
  }
}

```

### pint:standardize_not_equals
```
{
  "preset": "laravel",
  "rules": {
    "standardize_not_equals": true
  }
}

```

### pint:statement_indentation
```
{
  "preset": "laravel",
  "rules": {
    "statement_indentation": true
  }
}

```

### pint:strict_comparison
```
{
  "preset": "laravel",
  "rules": {
    "strict_comparison": true
  }
}

```

### pint:strict_param
```
{
  "preset": "laravel",
  "rules": {
    "strict_param": true
  }
}

```

### pint:switch_case_semicolon_to_colon
```
{
  "preset": "laravel",
  "rules": {
    "switch_case_semicolon_to_colon": true
  }
}

```

### pint:switch_case_space
```
{
  "preset": "laravel",
  "rules": {
    "switch_case_space": true
  }
}

```

### pint:switch_continue_to_break
```
{
  "preset": "symfony",
  "rules": {
    "switch_continue_to_break": true
  }
}

```

### pint:ternary_operator_spaces
```
{
  "preset": "laravel",
  "rules": {
    "ternary_operator_spaces": true
  }
}

```

### pint:ternary_to_null_coalescing
```
{
  "preset": "laravel",
  "rules": {
    "ternary_to_null_coalescing": true
  }
}

```

### pint:trailing_comma_in_multiline
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "trailing_comma_in_multiline": {
      "elements": [
        "arrays"
      ]
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "trailing_comma_in_multiline": {
      "elements": [
        "arguments",
        "arrays",
        "match",
        "parameters"
      ]
    }
  }
}
```

Alternative 3:
```json
{
  "preset": "laravel",
  "rules": {
    "trailing_comma_in_multiline": {
      "elements": [
        "arrays",
        "arguments",
        "parameters"
      ]
    }
  }
}
```

```

### pint:trim_array_spaces
```
{
  "preset": "laravel",
  "rules": {
    "trim_array_spaces": true
  }
}

```

### pint:type_declaration_spaces
```
{
  "preset": "laravel",
  "rules": {
    "type_declaration_spaces": true
  }
}

```

### pint:types_spaces
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "types_spaces": true
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "types_spaces": {
      "space": "none"
    }
  }
}
```

Alternative 3:
```json
{
  "preset": "laravel",
  "rules": {
    "types_spaces": {
      "space": "single"
    }
  }
}
```

```

### pint:unary_operator_spaces
```
{
  "preset": "laravel",
  "rules": {
    "unary_operator_spaces": true
  }
}

```

### pint:whitespace_after_comma_in_array
```
{
  "preset": "laravel",
  "rules": {
    "whitespace_after_comma_in_array": true
  }
}

```

### pint:yoda_style
```
Choose the pint.json block matching the practice/project policy; each block is a complete alternative.

Alternative 1:
```json
{
  "preset": "laravel",
  "rules": {
    "yoda_style": {
      "always_move_variable": false,
      "equal": false,
      "identical": false,
      "less_and_greater": false
    }
  }
}
```

Alternative 2:
```json
{
  "preset": "laravel",
  "rules": {
    "yoda_style": {
      "equal": false,
      "identical": false,
      "less_and_greater": false
    }
  }
}
```

Alternative 3:
```json
{
  "preset": "laravel",
  "rules": {
    "yoda_style": {
      "equal": true,
      "identical": true,
      "less_and_greater": false
    }
  }
}
```

```

### rector:AbortIfRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\If_\AbortIfRector::class]);

```

### rector:AddArgumentDefaultValueRector
```
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use RectorLaravel\Rector\ClassMethod\AddArgumentDefaultValueRector;
use RectorLaravel\ValueObject\AddArgumentDefaultValue;

return static function (RectorConfig $rectorConfig): void {


    $rectorConfig->ruleWithConfiguration(AddArgumentDefaultValueRector::class, [
        new AddArgumentDefaultValue('Illuminate\Contracts\Events\Dispatcher', 'listen', 1, null),
    ]);
};

```

### rector:AddExtendsAnnotationToModelFactoriesRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\AddExtendsAnnotationToModelFactoriesRector::class]);

```

### rector:AddGenericBuilderToScopesRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ClassMethod\AddGenericBuilderToScopesRector::class]);

```

### rector:AddGenericReturnTypeToRelationsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ClassMethod\AddGenericReturnTypeToRelationsRector::class]);

```

### rector:AddGuardToLoginEventRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\New_\AddGuardToLoginEventRector::class]);

```

### rector:AddHasFactoryToModelsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\AddHasFactoryToModelsRector::class]);

```

### rector:AddNewGenericReturnTypeToRelationsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ClassMethod\AddNewGenericReturnTypeToRelationsRector::class]);

```

### rector:AddParamTypeDeclarationRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel70.php',
    ]);

```

### rector:AddParentBootToModelClassMethodRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ClassMethod\AddParentBootToModelClassMethodRector::class]);

```

### rector:AddParentRegisterToEventServiceProviderRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ClassMethod\AddParentRegisterToEventServiceProviderRector::class]);

```

### rector:AddReturnTypeDeclarationRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel58.php',
    ]);

```

### rector:AddVoidReturnTypeWhereNoReturnRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\Rector\TypeDeclaration\Rector\ClassMethod\AddVoidReturnTypeWhereNoReturnRector::class]);

```

### rector:AliasesPropertyToAliasesAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\AliasesPropertyToAliasesAttributeRector::class]);

```

### rector:AnonymousMigrationsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\AnonymousMigrationsRector::class]);

```

### rector:AppEnvironmentComparisonToParameterRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Expr\AppEnvironmentComparisonToParameterRector::class]);

```

### rector:AppToResolveRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\AppToResolveRector::class]);

```

### rector:AppendsPropertyToAppendsAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\AppendsPropertyToAppendsAttributeRector::class]);

```

### rector:ApplyDefaultInsteadOfNullCoalesceRector
```
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use RectorLaravel\Rector\Coalesce\ApplyDefaultInsteadOfNullCoalesceRector;

return static function (RectorConfig $rectorConfig): void {


    $rectorConfig->rule(ApplyDefaultInsteadOfNullCoalesceRector::class);
};

```

### rector:ArgumentAdderRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel57.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel60.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel70.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel90.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel80.php',
    ]);

```

### rector:ArgumentFuncCallToMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([__DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel-static-to-injection.php']);

```

### rector:ArrayDimFetchToMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel-arrayaccess-to-method-call.php',
    ]);

```

### rector:ArrayToArrGetRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ArrayDimFetch\ArrayToArrGetRector::class]);

```

### rector:ArrayToFirstClassCallableRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\Rector\Php81\Rector\Array_\ArrayToFirstClassCallableRector::class]);

```

### rector:AssertSeeToAssertSeeHtmlRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\AssertSeeToAssertSeeHtmlRector::class]);

```

### rector:AssertStatusToAssertMethodRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\AssertStatusToAssertMethodRector::class]);

```

### rector:AssertWithClassStringToTypeHintedClosureRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\StaticCall\AssertWithClassStringToTypeHintedClosureRector::class]);

```

### rector:AvoidNegatedCollectionContainsOrDoesntContainRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\BooleanNot\AvoidNegatedCollectionContainsOrDoesntContainRector::class]);

```

### rector:AvoidNegatedCollectionFilterOrRejectRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\AvoidNegatedCollectionFilterOrRejectRector::class]);

```

### rector:BackoffPropertyToBackoffAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\BackoffPropertyToBackoffAttributeRector::class]);

```

### rector:CallOnAppArrayAccessToStandaloneAssignRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Assign\CallOnAppArrayAccessToStandaloneAssignRector::class]);

```

### rector:CarbonSetTestNowToTravelToRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\StaticCall\CarbonSetTestNowToTravelToRector::class]);

```

### rector:CarbonToDateFacadeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\StaticCall\CarbonToDateFacadeRector::class]);

```

### rector:CashierStripeOptionsToStripeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\CashierStripeOptionsToStripeRector::class]);

```

### rector:ChangeMethodVisibilityRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel56.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel60.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel90.php',
    ]);

```

### rector:ChangeNestedIfsToEarlyReturnRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\Rector\EarlyReturn\Rector\If_\ChangeNestedIfsToEarlyReturnRector::class]);

```

### rector:ChangeQueryWhereDateValueWithCarbonRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\ChangeQueryWhereDateValueWithCarbonRector::class]);

```

### rector:ClassPropertyAssignToConstructorPromotionRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\Rector\Php80\Rector\Class_\ClassPropertyAssignToConstructorPromotionRector::class]);

```

### rector:CollectedByPropertyToCollectedByAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\CollectedByPropertyToCollectedByAttributeRector::class]);

```

### rector:CollectsPropertyToCollectsAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\CollectsPropertyToCollectsAttributeRector::class]);

```

### rector:CommandHiddenPropertyToHiddenAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\CommandHiddenPropertyToHiddenAttributeRector::class]);

```

### rector:ConfigToTypedConfigMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\ConfigToTypedConfigMethodCallRector::class]);

```

### rector:ConnectionPropertyToConnectionAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\ConnectionPropertyToConnectionAttributeRector::class]);

```

### rector:ContainerBindConcreteWithClosureOnlyRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\ContainerBindConcreteWithClosureOnlyRector::class]);

```

### rector:ConvertEnumerableToArrayToAllRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\ConvertEnumerableToArrayToAllRector::class]);

```

### rector:DatabaseExpressionCastsToMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Cast\DatabaseExpressionCastsToMethodCallRector::class]);

```

### rector:DatabaseExpressionToStringToMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\DatabaseExpressionToStringToMethodCallRector::class]);

```

### rector:DateFormatPropertyToDateFormatAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\DateFormatPropertyToDateFormatAttributeRector::class]);

```

### rector:DateWhereClauseToShorthandRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\DateWhereClauseToShorthandRector::class]);

```

### rector:DelayPropertyToDelayAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\DelayPropertyToDelayAttributeRector::class]);

```

### rector:DeleteWhenMissingModelsPropertyToDeleteWhenMissingModelsAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\DeleteWhenMissingModelsPropertyToDeleteWhenMissingModelsAttributeRector::class]);

```

### rector:DescriptionPropertyToDescriptionAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\DescriptionPropertyToDescriptionAttributeRector::class]);

```

### rector:DispatchNonShouldQueueToDispatchSyncRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\DispatchNonShouldQueueToDispatchSyncRector::class]);

```

### rector:DispatchToHelperFunctionsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\StaticCall\DispatchToHelperFunctionsRector::class]);

```

### rector:EloquentMagicMethodToQueryBuilderRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\StaticCall\EloquentMagicMethodToQueryBuilderRector::class]);

```

### rector:EloquentOrderByToLatestOrOldestRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\EloquentOrderByToLatestOrOldestRector::class]);

```

### rector:EloquentWhereRelationTypeHintingParameterRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\EloquentWhereRelationTypeHintingParameterRector::class]);

```

### rector:EloquentWhereTypeHintClosureParameterRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\EloquentWhereTypeHintClosureParameterRector::class]);

```

### rector:EmptyGuardedPropertyToUnguardedAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\EmptyGuardedPropertyToUnguardedAttributeRector::class]);

```

### rector:EmptyToBlankAndFilledFuncRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Empty_\EmptyToBlankAndFilledFuncRector::class]);

```

### rector:EnvVariableToEnvHelperRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ArrayDimFetch\EnvVariableToEnvHelperRector::class]);

```

### rector:ErrorBagPropertyToErrorBagAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\ErrorBagPropertyToErrorBagAttributeRector::class]);

```

### rector:EventStringToClassConstantRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel52.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel54.php',
    ]);

```

### rector:FactoryApplyingStatesRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\FactoryApplyingStatesRector::class]);

```

### rector:FactoryDefinitionRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Namespace_\FactoryDefinitionRector::class]);

```

### rector:FactoryFuncCallToStaticCallRector
```
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use RectorLaravel\Rector\FuncCall\FactoryFuncCallToStaticCallRector;

return static function (RectorConfig $rectorConfig): void {


    $rectorConfig->rule(FactoryFuncCallToStaticCallRector::class);
};

```

### rector:FailOnTimeoutPropertyToFailOnTimeoutAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\FailOnTimeoutPropertyToFailOnTimeoutAttributeRector::class]);

```

### rector:FillablePropertyToFillableAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\FillablePropertyToFillableAttributeRector::class]);

```

### rector:FuncCallToNewRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel-static-to-injection.php',
    ]);

```

### rector:FuncCallToStaticCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel-array-str-functions-to-static-call.php',
    ]);

```

### rector:GuardedPropertyToGuardedAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\GuardedPropertyToGuardedAttributeRector::class]);

```

### rector:HelpPropertyToHelpAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\HelpPropertyToHelpAttributeRector::class]);

```

### rector:HelperFuncCallToFacadeClassRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\HelperFuncCallToFacadeClassRector::class]);

```

### rector:HiddenPropertyToHiddenAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\HiddenPropertyToHiddenAttributeRector::class]);

```

### rector:JobConnectionPropertyToJobConnectionAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\JobConnectionPropertyToJobConnectionAttributeRector::class]);

```

### rector:JsonCallToExplicitJsonCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\JsonCallToExplicitJsonCallRector::class]);

```

### rector:LivewireComponentComputedMethodToComputedAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\LivewireComponentComputedMethodToComputedAttributeRector::class]);

```

### rector:LivewireComponentQueryStringToUrlAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\LivewireComponentQueryStringToUrlAttributeRector::class]);

```

### rector:MakeModelAttributesAndScopesProtectedRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ClassMethod\MakeModelAttributesAndScopesProtectedRector::class]);

```

### rector:MaxExceptionsPropertyToMaxExceptionsAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\MaxExceptionsPropertyToMaxExceptionsAttributeRector::class]);

```

### rector:MigrateToSimplifiedAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ClassMethod\MigrateToSimplifiedAttributeRector::class]);

```

### rector:MinutesToSecondsInCacheRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\StaticCall\MinutesToSecondsInCacheRector::class]);

```

### rector:ModelCastsPropertyToCastsMethodRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\ModelCastsPropertyToCastsMethodRector::class]);

```

### rector:NewMethodCallWithoutParenthesesRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withPhpVersion(\Rector\ValueObject\PhpVersion::PHP_84)
    ->withRules([\Rector\Php84\Rector\MethodCall\NewMethodCallWithoutParenthesesRector::class]);

```

### rector:NotFilledBlankFuncCallToBlankFilledFuncCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\NotFilledBlankFuncCallToBlankFilledFuncCallRector::class]);

```

### rector:NowFuncWithStartOfDayMethodCallToTodayFuncRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\NowFuncWithStartOfDayMethodCallToTodayFuncRector::class]);

```

### rector:OptionalToNullsafeOperatorRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\PropertyFetch\OptionalToNullsafeOperatorRector::class]);

```

### rector:PropertyDeferToDeferrableProviderToRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\PropertyDeferToDeferrableProviderToRector::class]);

```

### rector:QueuePropertyToQueueAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\QueuePropertyToQueueAttributeRector::class]);

```

### rector:ReadOnlyPropertyRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\Rector\Php81\Rector\Property\ReadOnlyPropertyRector::class]);

```

### rector:Redirect301ToPermanentRedirectRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\StaticCall\Redirect301ToPermanentRedirectRector::class]);

```

### rector:RedirectBackToBackHelperRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\RedirectBackToBackHelperRector::class]);

```

### rector:RedirectRouteToToRouteHelperRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\RedirectRouteToToRouteHelperRector::class]);

```

### rector:RefactorBlueprintGeometryColumnsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\RefactorBlueprintGeometryColumnsRector::class]);

```

### rector:RemoveDumpDataDeadCodeRector
```
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use RectorLaravel\Rector\FuncCall\RemoveDumpDataDeadCodeRector;

return static function (RectorConfig $rectorConfig): void {


    $rectorConfig->ruleWithConfiguration(RemoveDumpDataDeadCodeRector::class, ['dd', 'dump']);
};

```

### rector:RemoveRedundantValueCallsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\RemoveRedundantValueCallsRector::class]);

```

### rector:RemoveRedundantWithCallsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\RemoveRedundantWithCallsRector::class]);

```

### rector:RemoveUnusedVariableInCatchRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\Rector\Php80\Rector\Catch_\RemoveUnusedVariableInCatchRector::class]);

```

### rector:RenameAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/packages/livewire/livewire-30.php',
    ]);

```

### rector:RenameClassRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel-facade-aliases-to-full-names.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel130-core.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel50.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel51.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel52.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel53.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel54.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel55.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel60.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel70.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/packages/livewire/livewire-40.php',
    ]);

```

### rector:RenameFunctionRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel100.php',
    ]);

```

### rector:RenameMethodRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel100.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel120.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel50.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel53.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel54.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel55.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel56.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel60.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel70.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel80.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel90.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/packages/cashier/cashier-13.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/packages/cashier/cashier-14.php',
    ]);

```

### rector:RenamePropertyRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel100.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel55.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel58.php',
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel80.php',
    ]);

```

### rector:ReplaceExpectsMethodsInTestsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\ReplaceExpectsMethodsInTestsRector::class]);

```

### rector:ReplaceFakerInstanceWithHelperRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\PropertyFetch\ReplaceFakerInstanceWithHelperRector::class]);

```

### rector:ReplaceFakerPropertyFetchWithMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\PropertyFetch\ReplaceFakerPropertyFetchWithMethodCallRector::class]);

```

### rector:ReplaceQueueTraitsWithQueueableRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\ReplaceQueueTraitsWithQueueableRector::class]);

```

### rector:ReplaceServiceContainerCallArgRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([__DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel-container-string-to-fully-qualified-name.php']);

```

### rector:ReplaceWithoutJobsEventsAndNotificationsWithFacadeFakeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\ReplaceWithoutJobsEventsAndNotificationsWithFacadeFakeRector::class]);

```

### rector:RequestStaticValidateToInjectRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\StaticCall\RequestStaticValidateToInjectRector::class]);

```

### rector:RequestVariablesToRequestFacadeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ArrayDimFetch\RequestVariablesToRequestFacadeRector::class]);

```

### rector:ResponseHelperCallToJsonResponseRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\ResponseHelperCallToJsonResponseRector::class]);

```

### rector:ReverseConditionableMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\ReverseConditionableMethodCallRector::class]);

```

### rector:RouteActionCallableRector
```
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use RectorLaravel\Rector\StaticCall\RouteActionCallableRector;

return static function (RectorConfig $rectorConfig): void {

    $rectorConfig->ruleWithConfiguration(RouteActionCallableRector::class, [
        RouteActionCallableRector::NAMESPACE => 'App\Http\Controllers',
    ]);
};

```

### rector:RouteKeyMethodToRouteKeyAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\RouteKeyMethodToRouteKeyAttributeRector::class]);

```

### rector:ScopeNamedClassMethodToScopeAttributedClassMethodRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ClassMethod\ScopeNamedClassMethodToScopeAttributedClassMethodRector::class]);

```

### rector:ServerVariableToRequestFacadeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ArrayDimFetch\ServerVariableToRequestFacadeRector::class]);

```

### rector:SessionVariableToSessionFacadeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\ArrayDimFetch\SessionVariableToSessionFacadeRector::class]);

```

### rector:SignaturePropertyToSignatureAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\SignaturePropertyToSignatureAttributeRector::class]);

```

### rector:SleepFuncToSleepStaticCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\SleepFuncToSleepStaticCallRector::class]);

```

### rector:StaticCallToFuncCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel53.php',
    ]);

```

### rector:StaticCallToMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withSets([
        __DIR__ . '/vendor/driftingly/rector-laravel/config/sets/laravel-static-to-injection.php',
    ]);

```

### rector:StopOnFirstFailurePropertyToStopOnFirstFailureAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\StopOnFirstFailurePropertyToStopOnFirstFailureAttributeRector::class]);

```

### rector:SubStrToStartsWithOrEndsWithStaticMethodCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Expr\SubStrToStartsWithOrEndsWithStaticMethodCallRector\SubStrToStartsWithOrEndsWithStaticMethodCallRector::class]);

```

### rector:TablePropertyToTableAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\TablePropertyToTableAttributeRector::class]);

```

### rector:ThrowIfAndThrowUnlessExceptionsToUseClassStringRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\ThrowIfAndThrowUnlessExceptionsToUseClassStringRector::class]);

```

### rector:ThrowIfRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\If_\ThrowIfRector::class]);

```

### rector:TimeoutPropertyToTimeoutAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\TimeoutPropertyToTimeoutAttributeRector::class]);

```

### rector:TriesPropertyToTriesAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\TriesPropertyToTriesAttributeRector::class]);

```

### rector:TypeHintTappableCallRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\TypeHintTappableCallRector::class]);

```

### rector:UnaliasCollectionMethodsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\UnaliasCollectionMethodsRector::class]);

```

### rector:UnifyModelDatesWithCastsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\UnifyModelDatesWithCastsRector::class]);

```

### rector:UniqueForPropertyToUniqueForAttributeRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\UniqueForPropertyToUniqueForAttributeRector::class]);

```

### rector:UnlinkFuncCallToFileFacadeDeleteRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\FuncCall\UnlinkFuncCallToFileFacadeDeleteRector::class]);

```

### rector:UseComponentPropertyWithinCommandsRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\UseComponentPropertyWithinCommandsRector::class]);

```

### rector:UseForwardsCallsTraitRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\Class_\UseForwardsCallsTraitRector::class]);

```

### rector:ValidationRuleArrayStringValueToArrayRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\ValidationRuleArrayStringValueToArrayRector::class]);

```

### rector:WhereNullComparisonToWhereNullRector
```
<?php

return \Rector\Config\RectorConfig::configure()
    ->withRules([\RectorLaravel\Rector\MethodCall\WhereNullComparisonToWhereNullRector::class]);

```

### rector:WhereToWhereLikeRector
```
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use RectorLaravel\Rector\MethodCall\WhereToWhereLikeRector;

return static function (RectorConfig $rectorConfig): void {


    $rectorConfig->rule(WhereToWhereLikeRector::class);
};

```

### shift:ci-generator/dusk-job
```
Shift product/task selection (no published per-rule PHP configuration):
Run the CI Generator Shift: https://laravelshift.com/ci-generator

```

### shift:laravel-linter/leverage-features
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Laravel Linter Shift: https://laravelshift.com/opinionated-laravel-way-shift

```

### shift:livewire-4/volt-to-sfc
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Livewire 4.x Shift Shift: https://laravelshift.com/upgrade-livewire-3-to-livewire-4

```

### shift:livewire-mfc-converter
```
Shift product/task selection (no published per-rule PHP configuration):
Run the MFC Converter Shift: https://laravelshift.com/convert-livewire-components-to-multi-file-components

```

### shift:lumen-to-laravel
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Lumen to Laravel Shift: https://laravelshift.com/convert-lumen-to-laravel

```

### shift:pest-converter/datasets
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Pest Converter Shift: https://laravelshift.com/phpunit-to-pest-converter

```

### shift:pest-converter/expectations
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Pest Converter Shift: https://laravelshift.com/phpunit-to-pest-converter

```

### shift:phpunit-13/type-specific-assertions
```
Shift product/task selection (no published per-rule PHP configuration):
Run the PHPUnit 13 Shift Shift: https://laravelshift.com/upgrade-phpunit-13

```

### shift:phpunit/attributes
```
Shift product/task selection (no published per-rule PHP configuration):
Run the PHPUnit 12 Shift Shift: https://laravelshift.com/upgrade-phpunit-12

```

### shift:phpunit/final-test-classes
```
Shift product/task selection (no published per-rule PHP configuration):
Run the PHPUnit 12 Shift Shift: https://laravelshift.com/upgrade-phpunit-12

```

### shift:preshift/abandoned-dependencies
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Laravel PreShift Shift: https://laravelshift.com/upgrade-laravel-guide-preshift

```

### shift:public/ci/composer-cache
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: CI Generator. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/ci/dusk
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: CI Generator. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/ci/php-environment
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: CI Generator. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/ci/private-packages
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: CI Generator. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/ci/style
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: CI Generator. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/ci/syntax
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: CI Generator. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/ci/tests
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: CI Generator. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/cloud/database
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Cloud Pre-Check. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/cloud/extensions
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Cloud Pre-Check. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/cloud/octane
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Cloud Pre-Check. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/cloud/php
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Cloud Pre-Check. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/cloud/system-commands
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Cloud Pre-Check. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/cloud/system-paths
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Cloud Pre-Check. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/fast-laravel/routes-file
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Fast Laravel. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/fast-laravel/static-group
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Fast Laravel. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/fixer/redundant-docblock-types
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Fixer (LF). Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/laravel13/config-env
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel 13.x Shift. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/laravel13/deprecations
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel 13.x Shift. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/laravel13/request-input
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel 13.x Shift. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/linter/composition
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Linter (LL). Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/linter/current-version
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Linter (LL). Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/linter/unused-directories
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Linter (LL). Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/namespace-models/move
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Namespace Models. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/slimmer/bootstrap
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Slimmer. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/slimmer/configuration
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Slimmer. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/slimmer/core-files
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Slimmer. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/slimmer/middleware
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Slimmer. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/slimmer/providers
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Slimmer. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/slimmer/scheduler
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Laravel Slimmer. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/vite/environment
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Vite Converter. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/vite/postcss
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Vite Converter. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:public/vite/test-helpers
```
Shift product/task selection (no published per-rule PHP configuration):
Public product: Vite Converter. Select this product in Shift; this check has no published per-rule configuration or internal rule ID.

```

### shift:slimmer/bootstrap
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Laravel Slimmer Shift: https://laravelshift.com/slim-laravel-application

```

### shift:upgrade-12/base-controller-traits
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Laravel 12.x Shift Shift: https://laravelshift.com/upgrade-laravel-11-to-laravel-12

```

### shift:upgrade-12/controller-middleware
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Laravel 12.x Shift Shift: https://laravelshift.com/upgrade-laravel-11-to-laravel-12

```

### shift:upgrade-9/lang-folder
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Laravel 9.x Shift Shift: https://laravelshift.com/upgrade-laravel-8-to-laravel-9

```

### shift:upgrade-9/swiftmailer
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Laravel 9.x Shift Shift: https://laravelshift.com/upgrade-laravel-8-to-laravel-9

```

### shift:vite-converter
```
Shift product/task selection (no published per-rule PHP configuration):
Run the Vite Converter Shift: https://laravelshift.com/convert-laravel-mix-to-vite

```

### shift:workbench/adopt-anonymous-migrations
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-anonymous-migrations",
  "catalog_id": 41
}

```

### shift:workbench/adopt-array-unpacking
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-array-unpacking",
  "catalog_id": 10
}

```

### shift:workbench/adopt-attribute-accessors-mutators
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-attribute-accessors-mutators",
  "catalog_id": 45
}

```

### shift:workbench/adopt-class-based-routes
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-class-based-routes",
  "catalog_id": 37
}

```

### shift:workbench/adopt-common-helpers
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-common-helpers",
  "catalog_id": 13
}

```

### shift:workbench/adopt-fluent-responses
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-fluent-responses",
  "catalog_id": 31
}

```

### shift:workbench/adopt-fluent-routes
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-fluent-routes",
  "catalog_id": 32
}

```

### shift:workbench/adopt-if-helpers
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-if-helpers",
  "catalog_id": 46,
  "ui_options": {
    "use-unless": false
  }
}

```

### shift:workbench/adopt-new-in-initializers
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-new-in-initializers",
  "catalog_id": 11
}

```

### shift:workbench/adopt-short-arrays
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-short-arrays",
  "catalog_id": 2
}

```

### shift:workbench/adopt-stringables
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-stringables",
  "catalog_id": 44,
  "ui_options": {
    "use-helper": false
  }
}

```

### shift:workbench/adopt-validation-arrays
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "adopt-validation-arrays",
  "catalog_id": 17
}

```

### shift:workbench/blade/csrf
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "modernize-blade-directives",
  "catalog_id": 33
}

```

### shift:workbench/blade/method
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "modernize-blade-directives",
  "catalog_id": 33
}

```

### shift:workbench/bump-laravel-dependencies
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "bump-laravel-dependencies",
  "catalog_id": 56,
  "ui_options": {
    "version": "13.0"
  }
}

```

### shift:workbench/bump-laravel-package-dependencies
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "bump-laravel-package-dependencies",
  "catalog_id": 57,
  "ui_options": {
    "append": true,
    "version": "13.0"
  }
}

```

### shift:workbench/convert-controller-validation
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "convert-controller-validation",
  "catalog_id": 34
}

```

### shift:workbench/convert-dates-property
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "convert-dates-property",
  "catalog_id": 15
}

```

### shift:workbench/convert-env-to-config
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "convert-env-to-config",
  "catalog_id": 36
}

```

### shift:workbench/convert-optional-to-nullsafe
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "convert-optional-to-nullsafe",
  "catalog_id": 19
}

```

### shift:workbench/convert-validation-rules
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "convert-validation-rules",
  "catalog_id": 35
}

```

### shift:workbench/declare-strict-types
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "declare-strict-types",
  "catalog_id": 8,
  "ui_options": {
    "additional-paths": null
  }
}

```

### shift:workbench/expand-resource-routes
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "expand-resource-routes",
  "catalog_id": 43
}

```

### shift:workbench/fix-phpunit-methods
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "fix-phpunit-methods",
  "catalog_id": 6
}

```

### shift:workbench/format-test-cases
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "format-test-cases",
  "catalog_id": 9,
  "ui_options": {
    "case": "Snake",
    "marker": "Prefix"
  }
}

```

### shift:workbench/laravel-global-scope-attribute
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "laravel-global-scope-attribute",
  "catalog_id": 48
}

```

### shift:workbench/laravel-local-scope-attribute
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "laravel-local-scope-attribute",
  "catalog_id": 49
}

```

### shift:workbench/laravel-observedby-attribute
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "laravel-observedby-attribute",
  "catalog_id": 47
}

```

### shift:workbench/laravel-property-attributes
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "laravel-property-attributes",
  "catalog_id": 50
}

```

### shift:workbench/leverage-command-autoloading
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "leverage-command-autoloading",
  "catalog_id": 30
}

```

### shift:workbench/leverage-request-injection
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "leverage-request-injection",
  "catalog_id": 27
}

```

### shift:workbench/modernize-blade-directives
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "modernize-blade-directives",
  "catalog_id": 33
}

```

### shift:workbench/normalize-request-access
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "normalize-request-access",
  "catalog_id": 28,
  "ui_options": {
    "access": "Object",
    "method": "input"
  }
}

```

### shift:workbench/normalize-view-data
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "normalize-view-data",
  "catalog_id": 29
}

```

### shift:workbench/remove-down-migration
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "remove-down-migration",
  "catalog_id": 40
}

```

### shift:workbench/remove-redundant-table-name
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "remove-redundant-table-name",
  "catalog_id": 14
}

```

### shift:workbench/remove-unused-composer-dependencies
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "remove-unused-composer-dependencies",
  "catalog_id": 55
}

```

### shift:workbench/replace-global-facades
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "replace-global-facades",
  "catalog_id": 12
}

```

### shift:workbench/resolve-helper
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "resolve-helper",
  "catalog_id": 25
}

```

### shift:workbench/set-readonly-properties
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "set-readonly-properties",
  "catalog_id": 4
}

```

### shift:workbench/streamline-order-methods
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "streamline-order-methods",
  "catalog_id": 16
}

```

### shift:workbench/type-hints
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "type-hints",
  "catalog_id": 51
}

```

### shift:workbench/use-class-constant
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "use-class-constant",
  "catalog_id": 3
}

```

### shift:workbench/use-latest-oldest
```
Shift Workbench UI task selection (not a local PHP configuration):
{
  "interface": "Shift Workbench",
  "task": "use-latest-oldest",
  "catalog_id": 22
}

```
