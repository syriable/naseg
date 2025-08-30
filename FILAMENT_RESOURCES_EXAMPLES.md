# Filament v4 Admin Resources Examples

## Laravel 12 + Filament v4 Integration for Marketplace Administration

This document provides comprehensive examples of Filament v4 resources for managing the Fiverr-like marketplace platform.

---

## 1. User Management Resource

### UserResource.php

```php
<?php

declare(strict_types=1);

namespace App\Filament\Resources;

use App\Filament\Resources\UserResource\Pages;
use App\Filament\Resources\UserResource\RelationManagers;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Filament\Infolists;
use Filament\Infolists\Infolist;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;
use Modules\Auth\Models\User;
use Spatie\Permission\Models\Role;

/**
 * Filament resource for comprehensive user management.
 * 
 * Provides admin interface for managing marketplace users including:
 * - User profiles and verification status
 * - Role and permission management
 * - Seller level progression
 * - Account suspension and moderation
 * - Media gallery management
 * 
 * Integrates with spatie/laravel-permission for role-based access control
 * and spatie/laravel-medialibrary for profile and portfolio management.
 */
class UserResource extends Resource
{
    protected static ?string $model = User::class;

    protected static ?string $navigationIcon = 'heroicon-o-users';
    protected static ?string $navigationGroup = 'User Management';
    protected static ?int $navigationSort = 1;

    /**
     * Configure the form for user creation and editing.
     * 
     * Provides comprehensive user management interface with validation,
     * role assignment, and profile management capabilities.
     */
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\Section::make('Basic Information')
                    ->schema([
                        Forms\Components\TextInput::make('email')
                            ->email()
                            ->required()
                            ->unique(ignorable: fn ($record) => $record)
                            ->maxLength(255)
                            ->helperText('Primary email address for account access'),

                        Forms\Components\Select::make('user_type')
                            ->options([
                                'buyer' => 'Buyer Only',
                                'seller' => 'Seller Only',
                                'both' => 'Buyer & Seller',
                            ])
                            ->required()
                            ->native(false)
                            ->helperText('Account type determines available features'),

                        Forms\Components\Select::make('status')
                            ->options([
                                'active' => 'Active',
                                'suspended' => 'Suspended',
                                'banned' => 'Banned',
                            ])
                            ->required()
                            ->native(false)
                            ->default('active'),

                        Forms\Components\DateTimePicker::make('email_verified_at')
                            ->helperText('Leave empty for unverified accounts'),
                    ])
                    ->columns(2),

                Forms\Components\Section::make('Profile Information')
                    ->schema([
                        Forms\Components\TextInput::make('userProfile.first_name')
                            ->label('First Name')
                            ->maxLength(50)
                            ->helperText('User\'s given name'),

                        Forms\Components\TextInput::make('userProfile.last_name')
                            ->label('Last Name')
                            ->maxLength(50)
                            ->helperText('User\'s family name'),

                        Forms\Components\Textarea::make('userProfile.bio')
                            ->label('Biography')
                            ->maxLength(1000)
                            ->rows(4)
                            ->helperText('Personal or professional description'),

                        Forms\Components\Select::make('userProfile.country')
                            ->label('Country')
                            ->options(self::getCountryOptions())
                            ->searchable()
                            ->native(false),

                        Forms\Components\Select::make('userProfile.timezone')
                            ->label('Timezone')
                            ->options(self::getTimezoneOptions())
                            ->searchable()
                            ->native(false),

                        Forms\Components\TagsInput::make('userProfile.languages')
                            ->label('Languages')
                            ->helperText('Languages spoken by the user'),
                    ])
                    ->columns(2)
                    ->relationship('userProfile'),

                Forms\Components\Section::make('Seller Information')
                    ->schema([
                        Forms\Components\Select::make('sellerProfile.level')
                            ->label('Seller Level')
                            ->options([
                                'new' => 'New Seller',
                                'level_one' => 'Level 1 Seller',
                                'level_two' => 'Level 2 Seller',
                                'top_rated' => 'Top Rated Seller',
                            ])
                            ->native(false)
                            ->helperText('Current seller progression level'),

                        Forms\Components\TextInput::make('sellerProfile.response_rate')
                            ->label('Response Rate (%)')
                            ->numeric()
                            ->minValue(0)
                            ->maxValue(100)
                            ->suffix('%')
                            ->helperText('Percentage of messages responded to within 24 hours'),

                        Forms\Components\TextInput::make('sellerProfile.completion_rate')
                            ->label('Completion Rate (%)')
                            ->numeric()
                            ->minValue(0)
                            ->maxValue(100)
                            ->suffix('%')
                            ->helperText('Percentage of orders completed successfully'),

                        Forms\Components\TextInput::make('sellerProfile.orders_completed')
                            ->label('Orders Completed')
                            ->numeric()
                            ->minValue(0)
                            ->helperText('Total number of successfully completed orders'),

                        Forms\Components\TextInput::make('sellerProfile.average_rating')
                            ->label('Average Rating')
                            ->numeric()
                            ->minValue(0)
                            ->maxValue(5)
                            ->step(0.1)
                            ->helperText('Average customer rating (1-5 stars)'),

                        Forms\Components\Toggle::make('sellerProfile.identity_verified')
                            ->label('Identity Verified')
                            ->helperText('Whether seller has completed identity verification'),

                        Forms\Components\TagsInput::make('sellerProfile.skills')
                            ->label('Skills')
                            ->helperText('Professional skills and expertise areas'),
                    ])
                    ->columns(3)
                    ->relationship('sellerProfile')
                    ->visible(fn ($get) => in_array($get('user_type'), ['seller', 'both'])),

                Forms\Components\Section::make('Roles & Permissions')
                    ->schema([
                        Forms\Components\CheckboxList::make('roles')
                            ->relationship('roles', 'name')
                            ->options(Role::pluck('name', 'id'))
                            ->columns(3)
                            ->helperText('Assign platform roles for access control'),

                        Forms\Components\CheckboxList::make('permissions')
                            ->relationship('permissions', 'name')
                            ->options(\Spatie\Permission\Models\Permission::pluck('name', 'id'))
                            ->columns(3)
                            ->helperText('Direct permissions (not recommended - use roles instead)'),
                    ])
                    ->columns(1),
            ]);
    }

    /**
     * Configure the table for user listing and management.
     * 
     * Provides powerful filtering, search, and bulk action capabilities
     * for efficient user administration.
     */
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\ImageColumn::make('avatar')
                    ->getStateUsing(fn (User $record) => $record->getAvatarUrl('avatar-small'))
                    ->circular()
                    ->size(40),

                Tables\Columns\TextColumn::make('email')
                    ->searchable()
                    ->sortable()
                    ->copyable()
                    ->description(fn (User $record) => $record->userProfile?->first_name . ' ' . $record->userProfile?->last_name),

                Tables\Columns\BadgeColumn::make('user_type')
                    ->colors([
                        'primary' => 'buyer',
                        'success' => 'seller',
                        'warning' => 'both',
                    ])
                    ->formatStateUsing(fn (string $state): string => match ($state) {
                        'buyer' => 'Buyer',
                        'seller' => 'Seller',
                        'both' => 'Both',
                    }),

                Tables\Columns\BadgeColumn::make('status')
                    ->colors([
                        'success' => 'active',
                        'warning' => 'suspended',
                        'danger' => 'banned',
                    ])
                    ->formatStateUsing(fn (string $state): string => ucfirst($state)),

                Tables\Columns\BadgeColumn::make('sellerProfile.level')
                    ->label('Seller Level')
                    ->colors([
                        'gray' => 'new',
                        'primary' => 'level_one',
                        'warning' => 'level_two',
                        'success' => 'top_rated',
                    ])
                    ->formatStateUsing(fn (?string $state): string => match ($state) {
                        'new' => 'New',
                        'level_one' => 'Level 1',
                        'level_two' => 'Level 2',
                        'top_rated' => 'Top Rated',
                        default => 'N/A',
                    })
                    ->visible(fn () => request()->get('tableFilters.user_type.value') !== 'buyer'),

                Tables\Columns\TextColumn::make('sellerProfile.orders_completed')
                    ->label('Orders')
                    ->numeric()
                    ->sortable()
                    ->alignCenter()
                    ->visible(fn () => request()->get('tableFilters.user_type.value') !== 'buyer'),

                Tables\Columns\TextColumn::make('sellerProfile.average_rating')
                    ->label('Rating')
                    ->formatStateUsing(fn (?float $state): string => 
                        $state ? number_format($state, 1) . ' ★' : 'No rating'
                    )
                    ->alignCenter()
                    ->visible(fn () => request()->get('tableFilters.user_type.value') !== 'buyer'),

                Tables\Columns\IconColumn::make('email_verified_at')
                    ->label('Verified')
                    ->boolean()
                    ->falseIcon('heroicon-o-x-circle')
                    ->trueIcon('heroicon-o-check-circle')
                    ->falseColor('danger')
                    ->trueColor('success'),

                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable()
                    ->toggleable(isToggledHiddenByDefault: true),

                Tables\Columns\TextColumn::make('updated_at')
                    ->dateTime()
                    ->sortable()
                    ->toggleable(isToggledHiddenByDefault: true),
            ])
            ->filters([
                Tables\Filters\SelectFilter::make('user_type')
                    ->options([
                        'buyer' => 'Buyer Only',
                        'seller' => 'Seller Only',
                        'both' => 'Both',
                    ])
                    ->native(false),

                Tables\Filters\SelectFilter::make('status')
                    ->options([
                        'active' => 'Active',
                        'suspended' => 'Suspended',
                        'banned' => 'Banned',
                    ])
                    ->native(false),

                Tables\Filters\SelectFilter::make('seller_level')
                    ->relationship('sellerProfile', 'level')
                    ->options([
                        'new' => 'New Seller',
                        'level_one' => 'Level 1',
                        'level_two' => 'Level 2',
                        'top_rated' => 'Top Rated',
                    ])
                    ->native(false),

                Tables\Filters\Filter::make('email_verified')
                    ->toggle()
                    ->query(fn (Builder $query): Builder => $query->whereNotNull('email_verified_at')),

                Tables\Filters\Filter::make('identity_verified')
                    ->toggle()
                    ->query(fn (Builder $query): Builder => 
                        $query->whereHas('sellerProfile', fn ($q) => $q->where('identity_verified', true))
                    ),

                Tables\Filters\Filter::make('created_this_month')
                    ->toggle()
                    ->query(fn (Builder $query): Builder => $query->whereMonth('created_at', now()->month)),
            ])
            ->actions([
                Tables\Actions\ViewAction::make(),
                Tables\Actions\EditAction::make(),
                
                Tables\Actions\Action::make('impersonate')
                    ->icon('heroicon-o-user-circle')
                    ->color('warning')
                    ->visible(fn () => auth()->user()->hasPermission('impersonate_users'))
                    ->url(fn (User $record) => route('impersonate', $record))
                    ->openUrlInNewTab(),

                Tables\Actions\Action::make('suspend')
                    ->icon('heroicon-o-pause-circle')
                    ->color('warning')
                    ->requiresConfirmation()
                    ->action(fn (User $record) => $record->update(['status' => 'suspended']))
                    ->visible(fn (User $record) => $record->status === 'active'),

                Tables\Actions\Action::make('activate')
                    ->icon('heroicon-o-play-circle')
                    ->color('success')
                    ->requiresConfirmation()
                    ->action(fn (User $record) => $record->update(['status' => 'active']))
                    ->visible(fn (User $record) => $record->status !== 'active'),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                    
                    Tables\Actions\BulkAction::make('verify_email')
                        ->label('Verify Email')
                        ->icon('heroicon-o-check-circle')
                        ->color('success')
                        ->requiresConfirmation()
                        ->action(fn ($records) => 
                            $records->each(fn ($record) => 
                                $record->update(['email_verified_at' => now()])
                            )
                        ),

                    Tables\Actions\BulkAction::make('suspend')
                        ->label('Suspend Users')
                        ->icon('heroicon-o-pause-circle')
                        ->color('warning')
                        ->requiresConfirmation()
                        ->action(fn ($records) => 
                            $records->each(fn ($record) => 
                                $record->update(['status' => 'suspended'])
                            )
                        ),
                ]),
            ])
            ->emptyStateActions([
                Tables\Actions\CreateAction::make(),
            ])
            ->defaultSort('created_at', 'desc');
    }

    /**
     * Configure the infolist for detailed user viewing.
     * 
     * Provides comprehensive user information display including
     * profile details, statistics, and media galleries.
     */
    public static function infolist(Infolist $infolist): Infolist
    {
        return $infolist
            ->schema([
                Infolists\Components\Section::make('User Overview')
                    ->schema([
                        Infolists\Components\Split::make([
                            Infolists\Components\Grid::make(2)
                                ->schema([
                                    Infolists\Components\ImageEntry::make('avatar')
                                        ->getStateUsing(fn (User $record) => $record->getAvatarUrl('avatar-large'))
                                        ->circular()
                                        ->size(120),

                                    Infolists\Components\Grid::make(1)
                                        ->schema([
                                            Infolists\Components\TextEntry::make('email')
                                                ->copyable()
                                                ->size(Infolists\Components\TextEntry\TextEntrySize::Large),

                                            Infolists\Components\TextEntry::make('full_name')
                                                ->getStateUsing(fn (User $record) => 
                                                    trim($record->userProfile?->first_name . ' ' . $record->userProfile?->last_name)
                                                )
                                                ->size(Infolists\Components\TextEntry\TextEntrySize::Medium),

                                            Infolists\Components\BadgeEntry::make('user_type')
                                                ->colors([
                                                    'primary' => 'buyer',
                                                    'success' => 'seller',
                                                    'warning' => 'both',
                                                ]),

                                            Infolists\Components\BadgeEntry::make('status')
                                                ->colors([
                                                    'success' => 'active',
                                                    'warning' => 'suspended',
                                                    'danger' => 'banned',
                                                ]),
                                        ]),
                                ]),
                        ]),
                    ])
                    ->columns(1),

                Infolists\Components\Section::make('Profile Details')
                    ->schema([
                        Infolists\Components\TextEntry::make('userProfile.bio')
                            ->label('Biography')
                            ->prose(),

                        Infolists\Components\Grid::make(3)
                            ->schema([
                                Infolists\Components\TextEntry::make('userProfile.country')
                                    ->label('Country')
                                    ->placeholder('Not specified'),

                                Infolists\Components\TextEntry::make('userProfile.timezone')
                                    ->label('Timezone')
                                    ->placeholder('Not specified'),

                                Infolists\Components\TextEntry::make('userProfile.languages')
                                    ->label('Languages')
                                    ->formatStateUsing(fn ($state) => implode(', ', $state ?? []))
                                    ->placeholder('Not specified'),
                            ]),
                    ])
                    ->columns(1),

                Infolists\Components\Section::make('Seller Statistics')
                    ->schema([
                        Infolists\Components\Grid::make(4)
                            ->schema([
                                Infolists\Components\TextEntry::make('sellerProfile.level')
                                    ->label('Seller Level')
                                    ->badge()
                                    ->colors([
                                        'gray' => 'new',
                                        'primary' => 'level_one',
                                        'warning' => 'level_two',
                                        'success' => 'top_rated',
                                    ])
                                    ->formatStateUsing(fn (?string $state): string => match ($state) {
                                        'new' => 'New Seller',
                                        'level_one' => 'Level 1 Seller',
                                        'level_two' => 'Level 2 Seller',
                                        'top_rated' => 'Top Rated Seller',
                                        default => 'N/A',
                                    }),

                                Infolists\Components\TextEntry::make('sellerProfile.orders_completed')
                                    ->label('Orders Completed')
                                    ->numeric()
                                    ->suffix(' orders'),

                                Infolists\Components\TextEntry::make('sellerProfile.response_rate')
                                    ->label('Response Rate')
                                    ->suffix('%')
                                    ->formatStateUsing(fn (?float $state): string => 
                                        $state ? number_format($state, 1) . '%' : 'N/A'
                                    ),

                                Infolists\Components\TextEntry::make('sellerProfile.completion_rate')
                                    ->label('Completion Rate')
                                    ->suffix('%')
                                    ->formatStateUsing(fn (?float $state): string => 
                                        $state ? number_format($state, 1) . '%' : 'N/A'
                                    ),
                            ]),

                        Infolists\Components\Grid::make(2)
                            ->schema([
                                Infolists\Components\TextEntry::make('sellerProfile.average_rating')
                                    ->label('Average Rating')
                                    ->formatStateUsing(fn (?float $state): string => 
                                        $state ? number_format($state, 1) . ' ★' : 'No rating'
                                    ),

                                Infolists\Components\IconEntry::make('sellerProfile.identity_verified')
                                    ->label('Identity Verified')
                                    ->boolean()
                                    ->trueIcon('heroicon-o-shield-check')
                                    ->falseIcon('heroicon-o-shield-exclamation')
                                    ->trueColor('success')
                                    ->falseColor('warning'),
                            ]),

                        Infolists\Components\TextEntry::make('sellerProfile.skills')
                            ->label('Skills')
                            ->formatStateUsing(fn ($state) => implode(', ', $state ?? []))
                            ->placeholder('No skills specified'),
                    ])
                    ->columns(1)
                    ->visible(fn (User $record) => in_array($record->user_type, ['seller', 'both'])),

                Infolists\Components\Section::make('Account Information')
                    ->schema([
                        Infolists\Components\Grid::make(2)
                            ->schema([
                                Infolists\Components\TextEntry::make('created_at')
                                    ->label('Account Created')
                                    ->dateTime()
                                    ->since(),

                                Infolists\Components\TextEntry::make('email_verified_at')
                                    ->label('Email Verified')
                                    ->dateTime()
                                    ->placeholder('Not verified')
                                    ->since(),
                            ]),
                    ])
                    ->columns(1),
            ]);
    }

    /**
     * Get relation managers for user resource.
     */
    public static function getRelations(): array
    {
        return [
            RelationManagers\GigsRelationManager::class,
            RelationManagers\OrdersRelationManager::class,
            RelationManagers\MediaRelationManager::class,
        ];
    }

    /**
     * Get pages for user resource.
     */
    public static function getPages(): array
    {
        return [
            'index' => Pages\ListUsers::route('/'),
            'create' => Pages\CreateUser::route('/create'),
            'view' => Pages\ViewUser::route('/{record}'),
            'edit' => Pages\EditUser::route('/{record}/edit'),
        ];
    }

    /**
     * Get country options for profile form.
     */
    private static function getCountryOptions(): array
    {
        return [
            'US' => 'United States',
            'CA' => 'Canada',
            'GB' => 'United Kingdom',
            'AU' => 'Australia',
            'DE' => 'Germany',
            'FR' => 'France',
            'IT' => 'Italy',
            'ES' => 'Spain',
            'NL' => 'Netherlands',
            'SE' => 'Sweden',
            'NO' => 'Norway',
            'DK' => 'Denmark',
            'JP' => 'Japan',
            'KR' => 'South Korea',
            'IN' => 'India',
            'BR' => 'Brazil',
            'MX' => 'Mexico',
            'AR' => 'Argentina',
        ];
    }

    /**
     * Get timezone options for profile form.
     */
    private static function getTimezoneOptions(): array
    {
        return [
            'UTC' => 'UTC',
            'America/New_York' => 'Eastern Time (US)',
            'America/Chicago' => 'Central Time (US)',
            'America/Denver' => 'Mountain Time (US)',
            'America/Los_Angeles' => 'Pacific Time (US)',
            'Europe/London' => 'London',
            'Europe/Paris' => 'Paris',
            'Europe/Berlin' => 'Berlin',
            'Asia/Tokyo' => 'Tokyo',
            'Asia/Shanghai' => 'Shanghai',
            'Australia/Sydney' => 'Sydney',
        ];
    }
}
```

---

## 2. Order Management Resource

### OrderResource.php

```php
<?php

declare(strict_types=1);

namespace App\Filament\Resources;

use App\Filament\Resources\OrderResource\Pages;
use App\Filament\Resources\OrderResource\RelationManagers;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Filament\Infolists;
use Filament\Infolists\Infolist;
use Illuminate\Database\Eloquent\Builder;
use Modules\Orders\Models\Order;
use Modules\Orders\States;

/**
 * Filament resource for comprehensive order management.
 * 
 * Provides admin interface for monitoring and managing marketplace orders:
 * - Order lifecycle state tracking
 * - Payment and commission management
 * - Dispute resolution workflow
 * - Performance analytics
 * - Bulk order operations
 * 
 * Integrates with spatie/laravel-model-states for state management
 * and cknow/laravel-money for financial calculations.
 */
class OrderResource extends Resource
{
    protected static ?string $model = Order::class;

    protected static ?string $navigationIcon = 'heroicon-o-shopping-cart';
    protected static ?string $navigationGroup = 'Order Management';
    protected static ?int $navigationSort = 1;
    protected static ?string $recordTitleAttribute = 'order_number';

    /**
     * Configure order management form.
     * 
     * Allows admin modification of order details, state transitions,
     * and resolution actions. Includes financial calculations and
     * state-specific business logic.
     */
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\Section::make('Order Details')
                    ->schema([
                        Forms\Components\TextInput::make('order_number')
                            ->required()
                            ->unique(ignoreRecord: true)
                            ->maxLength(20)
                            ->disabled(fn ($context) => $context === 'edit'),

                        Forms\Components\Select::make('gig_id')
                            ->relationship('gig', 'title')
                            ->required()
                            ->searchable()
                            ->preload()
                            ->helperText('The gig this order is for'),

                        Forms\Components\Select::make('buyer_id')
                            ->relationship('buyer', 'email')
                            ->required()
                            ->searchable()
                            ->preload()
                            ->helperText('Customer placing the order'),

                        Forms\Components\Select::make('seller_id')
                            ->relationship('seller', 'email')
                            ->required()
                            ->searchable()
                            ->preload()
                            ->helperText('Seller fulfilling the order'),

                        Forms\Components\Select::make('package_id')
                            ->relationship('package', 'title')
                            ->required()
                            ->searchable()
                            ->helperText('Selected pricing package'),
                    ])
                    ->columns(2),

                Forms\Components\Section::make('Financial Information')
                    ->schema([
                        Forms\Components\TextInput::make('total_price_cents')
                            ->required()
                            ->numeric()
                            ->minValue(500) // Minimum $5.00
                            ->helperText('Total price in cents (e.g., 2500 for $25.00)')
                            ->formatStateUsing(fn ($state) => $state ? $state / 100 : null)
                            ->dehydrateStateUsing(fn ($state) => $state ? $state * 100 : null)
                            ->prefix('$'),

                        Forms\Components\Select::make('currency_code')
                            ->options([
                                'USD' => 'US Dollar',
                                'EUR' => 'Euro',
                                'GBP' => 'British Pound',
                                'CAD' => 'Canadian Dollar',
                                'AUD' => 'Australian Dollar',
                            ])
                            ->required()
                            ->default('USD')
                            ->native(false),

                        Forms\Components\Placeholder::make('fee_breakdown')
                            ->label('Fee Breakdown')
                            ->content(function ($get) {
                                $totalCents = (int) ($get('total_price_cents') ?? 0) * 100;
                                if ($totalCents <= 0) return 'Enter price to see breakdown';

                                $sellerCommission = (int) ($totalCents * 0.20);
                                $buyerFee = (int) ($totalCents * 0.055);
                                $sellerEarnings = $totalCents - $sellerCommission;

                                return sprintf(
                                    "Seller Commission: $%.2f\nBuyer Fee: $%.2f\nSeller Earnings: $%.2f",
                                    $sellerCommission / 100,
                                    $buyerFee / 100,
                                    $sellerEarnings / 100
                                );
                            })
                            ->columnSpanFull(),
                    ])
                    ->columns(2),

                Forms\Components\Section::make('Timeline & Dates')
                    ->schema([
                        Forms\Components\DateTimePicker::make('due_date')
                            ->helperText('Expected delivery date based on package terms'),

                        Forms\Components\DateTimePicker::make('delivered_at')
                            ->helperText('When seller submitted deliverables'),

                        Forms\Components\DateTimePicker::make('completed_at')
                            ->helperText('When order was marked complete'),
                    ])
                    ->columns(3),

                Forms\Components\Section::make('State Management')
                    ->schema([
                        Forms\Components\Select::make('state')
                            ->options([
                                States\OrderPlacedState::class => 'Order Placed',
                                States\OrderRequirementsSubmittedState::class => 'Requirements Submitted',
                                States\OrderInProgressState::class => 'In Progress',
                                States\OrderDeliveredState::class => 'Delivered',
                                States\OrderCompletedState::class => 'Completed',
                                States\OrderCancelledState::class => 'Cancelled',
                                States\OrderDisputedState::class => 'Disputed',
                                States\OrderLateState::class => 'Late',
                                States\OrderRevisionRequestedState::class => 'Revision Requested',
                            ])
                            ->required()
                            ->native(false)
                            ->helperText('Current order state - transitions may have business rules'),

                        Forms\Components\Textarea::make('admin_notes')
                            ->label('Admin Notes')
                            ->rows(3)
                            ->helperText('Internal notes for order management'),
                    ])
                    ->columns(1),
            ]);
    }

    /**
     * Configure order listing table.
     * 
     * Provides comprehensive order monitoring with filtering,
     * search, and bulk action capabilities for efficient management.
     */
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('order_number')
                    ->label('Order #')
                    ->searchable()
                    ->sortable()
                    ->copyable()
                    ->weight('bold'),

                Tables\Columns\TextColumn::make('gig.title')
                    ->label('Gig')
                    ->limit(30)
                    ->tooltip(function (Tables\Columns\TextColumn $column): ?string {
                        $state = $column->getState();
                        return strlen($state) > 30 ? $state : null;
                    })
                    ->searchable(),

                Tables\Columns\TextColumn::make('buyer.email')
                    ->label('Buyer')
                    ->searchable()
                    ->copyable()
                    ->formatStateUsing(fn ($state, $record) => 
                        $record->buyer?->userProfile?->first_name 
                            ? $record->buyer->userProfile->first_name . ' (' . $state . ')'
                            : $state
                    ),

                Tables\Columns\TextColumn::make('seller.email')
                    ->label('Seller')
                    ->searchable()
                    ->copyable()
                    ->formatStateUsing(fn ($state, $record) => 
                        $record->seller?->userProfile?->first_name 
                            ? $record->seller->userProfile->first_name . ' (' . $state . ')'
                            : $state
                    ),

                Tables\Columns\BadgeColumn::make('state')
                    ->colors([
                        'gray' => States\OrderPlacedState::class,
                        'info' => States\OrderRequirementsSubmittedState::class,
                        'warning' => States\OrderInProgressState::class,
                        'success' => States\OrderDeliveredState::class,
                        'success' => States\OrderCompletedState::class,
                        'danger' => States\OrderCancelledState::class,
                        'danger' => States\OrderDisputedState::class,
                        'danger' => States\OrderLateState::class,
                        'info' => States\OrderRevisionRequestedState::class,
                    ])
                    ->formatStateUsing(fn ($state) => class_basename($state))
                    ->sortable(),

                Tables\Columns\TextColumn::make('total_price_cents')
                    ->label('Total')
                    ->money('USD', divideBy: 100)
                    ->sortable()
                    ->alignEnd(),

                Tables\Columns\TextColumn::make('seller_earnings')
                    ->label('Seller Earnings')
                    ->getStateUsing(function (Order $record): int {
                        $fees = $record->calculateFees();
                        return $fees['seller_earnings']->getAmount();
                    })
                    ->money('USD', divideBy: 100)
                    ->alignEnd()
                    ->color('success'),

                Tables\Columns\IconColumn::make('is_overdue')
                    ->label('Overdue')
                    ->getStateUsing(fn (Order $record): bool => $record->isOverdue())
                    ->boolean()
                    ->trueIcon('heroicon-o-exclamation-triangle')
                    ->falseIcon('heroicon-o-check-circle')
                    ->trueColor('danger')
                    ->falseColor('success'),

                Tables\Columns\TextColumn::make('due_date')
                    ->label('Due')
                    ->dateTime()
                    ->sortable()
                    ->color(fn ($state) => $state && $state->isPast() ? 'danger' : 'gray')
                    ->since(),

                Tables\Columns\TextColumn::make('created_at')
                    ->label('Created')
                    ->dateTime()
                    ->sortable()
                    ->since(),
            ])
            ->filters([
                Tables\Filters\SelectFilter::make('state')
                    ->options([
                        States\OrderPlacedState::class => 'Placed',
                        States\OrderRequirementsSubmittedState::class => 'Requirements Submitted',
                        States\OrderInProgressState::class => 'In Progress',
                        States\OrderDeliveredState::class => 'Delivered',
                        States\OrderCompletedState::class => 'Completed',
                        States\OrderCancelledState::class => 'Cancelled',
                        States\OrderDisputedState::class => 'Disputed',
                        States\OrderLateState::class => 'Late',
                    ])
                    ->native(false)
                    ->multiple(),

                Tables\Filters\Filter::make('overdue')
                    ->toggle()
                    ->query(fn (Builder $query): Builder => 
                        $query->where(function ($q) {
                            $q->where('due_date', '<', now())
                              ->whereNotIn('state', [
                                  States\OrderCompletedState::class,
                                  States\OrderCancelledState::class,
                              ]);
                        })
                    ),

                Tables\Filters\Filter::make('high_value')
                    ->label('High Value ($500+)')
                    ->toggle()
                    ->query(fn (Builder $query): Builder => $query->where('total_price_cents', '>=', 50000)),

                Tables\Filters\Filter::make('created_this_week')
                    ->toggle()
                    ->query(fn (Builder $query): Builder => $query->whereBetween('created_at', [
                        now()->startOfWeek(),
                        now()->endOfWeek(),
                    ])),

                Tables\Filters\Filter::make('needs_attention')
                    ->label('Needs Attention')
                    ->toggle()
                    ->query(fn (Builder $query): Builder => 
                        $query->where(function ($q) {
                            $q->whereIn('state', [
                                States\OrderDisputedState::class,
                                States\OrderLateState::class,
                            ])
                            ->orWhere(function ($subQuery) {
                                $subQuery->where('state', States\OrderPlacedState::class)
                                         ->where('created_at', '<', now()->subDays(5));
                            });
                        })
                    ),
            ])
            ->actions([
                Tables\Actions\ViewAction::make(),
                Tables\Actions\EditAction::make(),

                Tables\Actions\Action::make('transition_state')
                    ->icon('heroicon-o-arrow-right-circle')
                    ->color('info')
                    ->form([
                        Forms\Components\Select::make('new_state')
                            ->label('Transition to State')
                            ->options([
                                States\OrderPlacedState::class => 'Order Placed',
                                States\OrderRequirementsSubmittedState::class => 'Requirements Submitted',
                                States\OrderInProgressState::class => 'In Progress',
                                States\OrderDeliveredState::class => 'Delivered',
                                States\OrderCompletedState::class => 'Completed',
                                States\OrderCancelledState::class => 'Cancelled',
                                States\OrderDisputedState::class => 'Disputed',
                            ])
                            ->required()
                            ->native(false),
                        
                        Forms\Components\Textarea::make('transition_reason')
                            ->label('Reason for State Change')
                            ->required()
                            ->rows(3),
                    ])
                    ->action(function (Order $record, array $data): void {
                        $record->state->transitionTo($data['new_state']);
                        
                        activity()
                            ->performedOn($record)
                            ->withProperties([
                                'new_state' => $data['new_state'],
                                'reason' => $data['transition_reason'],
                                'admin_user' => auth()->id(),
                            ])
                            ->log('Order state changed by admin');
                    }),

                Tables\Actions\Action::make('refund')
                    ->icon('heroicon-o-banknotes')
                    ->color('warning')
                    ->requiresConfirmation()
                    ->modalDescription('This will process a full refund and cancel the order.')
                    ->visible(fn (Order $record) => !in_array($record->state::class, [
                        States\OrderCompletedState::class,
                        States\OrderCancelledState::class,
                    ]))
                    ->action(function (Order $record): void {
                        // Process refund logic here
                        \Modules\Payments\Jobs\ProcessRefundJob::dispatch($record);
                        $record->state->transitionTo(States\OrderCancelledState::class);
                    }),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),

                    Tables\Actions\BulkAction::make('mark_disputed')
                        ->label('Mark as Disputed')
                        ->icon('heroicon-o-exclamation-triangle')
                        ->color('danger')
                        ->requiresConfirmation()
                        ->action(fn ($records) => 
                            $records->each(fn ($record) => 
                                $record->state->transitionTo(States\OrderDisputedState::class)
                            )
                        ),

                    Tables\Actions\BulkAction::make('send_reminder')
                        ->label('Send Reminder')
                        ->icon('heroicon-o-bell')
                        ->color('info')
                        ->action(fn ($records) => 
                            $records->each(fn ($record) => 
                                \Modules\Notifications\Jobs\SendOrderReminderJob::dispatch($record)
                            )
                        ),
                ]),
            ])
            ->emptyStateActions([
                Tables\Actions\CreateAction::make(),
            ])
            ->defaultSort('created_at', 'desc')
            ->poll('30s'); // Auto-refresh every 30 seconds
    }

    /**
     * Configure detailed order information display.
     * 
     * Provides comprehensive order overview including timeline,
     * financial breakdown, and related activities.
     */
    public static function infolist(Infolist $infolist): Infolist
    {
        return $infolist
            ->schema([
                Infolists\Components\Section::make('Order Overview')
                    ->schema([
                        Infolists\Components\Split::make([
                            Infolists\Components\Grid::make(2)
                                ->schema([
                                    Infolists\Components\TextEntry::make('order_number')
                                        ->label('Order Number')
                                        ->copyable()
                                        ->size(Infolists\Components\TextEntry\TextEntrySize::Large)
                                        ->weight('bold'),

                                    Infolists\Components\BadgeEntry::make('state')
                                        ->colors([
                                            'gray' => States\OrderPlacedState::class,
                                            'info' => States\OrderRequirementsSubmittedState::class,
                                            'warning' => States\OrderInProgressState::class,
                                            'success' => States\OrderDeliveredState::class,
                                            'success' => States\OrderCompletedState::class,
                                            'danger' => States\OrderCancelledState::class,
                                            'danger' => States\OrderDisputedState::class,
                                            'danger' => States\OrderLateState::class,
                                        ])
                                        ->formatStateUsing(fn ($state) => $state->getDisplayName()),

                                    Infolists\Components\TextEntry::make('gig.title')
                                        ->label('Gig Title'),

                                    Infolists\Components\TextEntry::make('package.title')
                                        ->label('Selected Package'),
                                ]),
                        ]),
                    ])
                    ->columns(1),

                Infolists\Components\Section::make('Participants')
                    ->schema([
                        Infolists\Components\Grid::make(2)
                            ->schema([
                                Infolists\Components\TextEntry::make('buyer.email')
                                    ->label('Buyer')
                                    ->formatStateUsing(fn ($state, $record) => 
                                        $record->buyer?->userProfile?->first_name 
                                            ? $record->buyer->userProfile->first_name . ' (' . $state . ')'
                                            : $state
                                    )
                                    ->copyable(),

                                Infolists\Components\TextEntry::make('seller.email')
                                    ->label('Seller')
                                    ->formatStateUsing(fn ($state, $record) => 
                                        $record->seller?->userProfile?->first_name 
                                            ? $record->seller->userProfile->first_name . ' (' . $state . ')'
                                            : $state
                                    )
                                    ->copyable(),
                            ]),
                    ])
                    ->columns(1),

                Infolists\Components\Section::make('Financial Breakdown')
                    ->schema([
                        Infolists\Components\Grid::make(3)
                            ->schema([
                                Infolists\Components\TextEntry::make('total_price_cents')
                                    ->label('Order Total')
                                    ->money('USD', divideBy: 100)
                                    ->size(Infolists\Components\TextEntry\TextEntrySize::Large)
                                    ->weight('bold'),

                                Infolists\Components\TextEntry::make('seller_commission')
                                    ->label('Platform Commission (20%)')
                                    ->getStateUsing(function (Order $record): int {
                                        $fees = $record->calculateFees();
                                        return $fees['seller_commission']->getAmount();
                                    })
                                    ->money('USD', divideBy: 100)
                                    ->color('warning'),

                                Infolists\Components\TextEntry::make('seller_earnings')
                                    ->label('Seller Earnings')
                                    ->getStateUsing(function (Order $record): int {
                                        $fees = $record->calculateFees();
                                        return $fees['seller_earnings']->getAmount();
                                    })
                                    ->money('USD', divideBy: 100)
                                    ->size(Infolists\Components\TextEntry\TextEntrySize::Large)
                                    ->color('success')
                                    ->weight('bold'),
                            ]),
                    ])
                    ->columns(1),

                Infolists\Components\Section::make('Timeline')
                    ->schema([
                        Infolists\Components\Grid::make(3)
                            ->schema([
                                Infolists\Components\TextEntry::make('created_at')
                                    ->label('Order Placed')
                                    ->dateTime()
                                    ->since(),

                                Infolists\Components\TextEntry::make('due_date')
                                    ->label('Due Date')
                                    ->dateTime()
                                    ->color(fn ($state) => $state && $state->isPast() ? 'danger' : 'gray')
                                    ->since()
                                    ->placeholder('Not set'),

                                Infolists\Components\TextEntry::make('completed_at')
                                    ->label('Completed')
                                    ->dateTime()
                                    ->since()
                                    ->placeholder('Not completed'),
                            ]),

                        Infolists\Components\TextEntry::make('time_remaining')
                            ->label('Time Remaining')
                            ->getStateUsing(fn (Order $record): ?string => $record->getTimeRemaining())
                            ->placeholder('No active deadline')
                            ->color(fn ($state) => str_contains($state ?? '', 'Overdue') ? 'danger' : 'success'),
                    ])
                    ->columns(1),

                Infolists\Components\Section::make('Order Requirements')
                    ->schema([
                        Infolists\Components\RepeatableEntry::make('requirements')
                            ->schema([
                                Infolists\Components\TextEntry::make('question')
                                    ->label('Question'),
                                Infolists\Components\TextEntry::make('answer')
                                    ->label('Answer')
                                    ->prose(),
                            ])
                            ->columns(2),
                    ])
                    ->columns(1)
                    ->visible(fn (Order $record) => $record->requirements()->exists()),

                Infolists\Components\Section::make('Deliverables')
                    ->schema([
                        Infolists\Components\RepeatableEntry::make('deliverables')
                            ->schema([
                                Infolists\Components\TextEntry::make('title')
                                    ->label('Deliverable'),
                                Infolists\Components\TextEntry::make('description')
                                    ->label('Description')
                                    ->prose(),
                                Infolists\Components\TextEntry::make('created_at')
                                    ->label('Submitted')
                                    ->dateTime()
                                    ->since(),
                            ])
                            ->columns(3),
                    ])
                    ->columns(1)
                    ->visible(fn (Order $record) => $record->deliverables()->exists()),
            ]);
    }

    /**
     * Get relation managers for order resource.
     */
    public static function getRelations(): array
    {
        return [
            RelationManagers\RequirementsRelationManager::class,
            RelationManagers\DeliverablesRelationManager::class,
            RelationManagers\RevisionsRelationManager::class,
            RelationManagers\MessagesRelationManager::class,
        ];
    }

    /**
     * Get pages for order resource.
     */
    public static function getPages(): array
    {
        return [
            'index' => Pages\ListOrders::route('/'),
            'create' => Pages\CreateOrder::route('/create'),
            'view' => Pages\ViewOrder::route('/{record}'),
            'edit' => Pages\EditOrder::route('/{record}/edit'),
        ];
    }

    /**
     * Global search configuration for orders.
     */
    public static function getGlobalSearchResultTitle(Model $record): string
    {
        return "Order {$record->order_number}";
    }

    /**
     * Global search result details.
     */
    public static function getGlobalSearchResultDetails(Model $record): array
    {
        return [
            'Buyer' => $record->buyer?->email,
            'Seller' => $record->seller?->email,
            'Total' => '$' . number_format($record->total_price_cents / 100, 2),
            'State' => class_basename($record->state),
        ];
    }

    /**
     * Global search result URL.
     */
    public static function getGlobalSearchResultUrl(Model $record): string
    {
        return OrderResource::getUrl('view', ['record' => $record]);
    }
}
```

---

This demonstrates comprehensive Filament v4 integration with:

1. **Complete CRUD Operations** with proper validation and relationships
2. **Advanced Filtering & Search** for efficient data management
3. **Custom Actions** for state transitions and business operations
4. **Financial Calculations** integrated with cknow/laravel-money
5. **State Management** visualization and control
6. **Bulk Operations** for administrative efficiency
7. **Detailed Information Views** with comprehensive data display
8. **Media Integration** through relation managers
9. **Access Control** through proper authorization
10. **Real-time Updates** with polling and notifications

The implementation follows Filament v4 best practices and integrates seamlessly with all mandated packages while maintaining type safety and comprehensive documentation.