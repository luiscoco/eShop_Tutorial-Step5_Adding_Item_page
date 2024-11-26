# Building 'eShop' from Zero to Hero: We add Item Page

## 1. We add the ItemPage razor component

We run Visual Studio 2022 and we open the solution downloaded in this github repo:

https://github.com/luiscoco/eShop_Aspire-Tutorial-Step4__IdentityServer4

We navigate to the WebApp project and we add a new folder **Item**, see this picture

![image](https://github.com/user-attachments/assets/f60d3140-76ab-48ad-8f65-24dfdf5ecb4b)

Inside the **Item** folder we create a new razor component called **ItemPage.razor**. This is the new componente source code:

```razor
@page "/item/{itemId:int}"
@using System.Net
@inject CatalogService CatalogService
@* @inject BasketState BasketState *@
@inject NavigationManager Nav
@inject IProductImageUrlProvider ProductImages

@if (item is not null)
{
    <PageTitle>@item.Name | AdventureWorks</PageTitle>
    <SectionContent SectionName="page-header-title">@item.Name</SectionContent>
    <SectionContent SectionName="page-header-subtitle">@item.CatalogBrand?.Brand</SectionContent>

    <div class="item-details">
        <img alt="@item.Name" src="@ProductImages.GetProductImageUrl(item)" />
        <div class="description">
            <p>@item.Description</p>
            <p>
                Brand: <strong>@item.CatalogBrand?.Brand</strong>
            </p>
            <form class="add-to-cart" method="post" @formname="add-to-cart" @onsubmit="@AddToCartAsync" data-enhance="@isLoggedIn">
                <AntiforgeryToken />
                <span class="price">$@item.Price.ToString("0.00")</span>

                @if (isLoggedIn)
                {
                    <button type="submit" title="Add to basket">
                        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" xmlns="http://www.w3.org/2000/svg">
                            <path id="Vector" d="M6 2L3 6V20C3 20.5304 3.21071 21.0391 3.58579 21.4142C3.96086 21.7893 4.46957 22 5 22H19C19.5304 22 20.0391 21.7893 20.4142 21.4142C20.7893 21.0391 21 20.5304 21 20V6L18 2H6Z" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
                            <path id="Vector_2" d="M3 6H21" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
                            <path id="Vector_3" d="M16 10C16 11.0609 15.5786 12.0783 14.8284 12.8284C14.0783 13.5786 13.0609 14 12 14C10.9391 14 9.92172 13.5786 9.17157 12.8284C8.42143 12.0783 8 11.0609 8 10" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
                        </svg>
                        Add to shopping bag
                    </button>
                }
                else
                {
                    <button type="submit" title="Log in to purchase">
                        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" xmlns="http://www.w3.org/2000/svg">
                            <path d="M20 21V19C20 17.9391 19.5786 16.9217 18.8284 16.1716C18.0783 15.4214 17.0609 15 16 15H8C6.93913 15 5.92172 15.4214 5.17157 16.1716C4.42143 16.9217 4 17.9391 4 19V21" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
                            <path d="M12 11C14.2091 11 16 9.20914 16 7C16 4.79086 14.2091 3 12 3C9.79086 3 8 4.79086 8 7C8 9.20914 9.79086 11 12 11Z" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
                        </svg>
                        Log in to purchase
                    </button>
                }
            </form>

            @if (numInCart > 0)
            {
                <p><strong>@numInCart</strong> in <a href="cart">shopping bag</a></p>
            }
        </div>
    </div>
}
else if (notFound)
{
    <SectionContent SectionName="page-header-title">Not found</SectionContent>
    <div class="item-details">
        <p>Sorry, we couldn't find any such product.</p>
    </div>
}

@code {
    private CatalogItem? item;
    private int numInCart;
    private bool isLoggedIn;
    private bool notFound;

    [Parameter]
    public int ItemId { get; set; }

    [CascadingParameter]
    public HttpContext? HttpContext { get; set; }

    protected override async Task OnInitializedAsync()
    {
        try
        {
            isLoggedIn = HttpContext?.User.Identity?.IsAuthenticated == true;
            item = await CatalogService.GetCatalogItem(ItemId);
            await UpdateNumInCartAsync();
        }
        catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
        {
            HttpContext!.Response.StatusCode = 404;
            notFound = true;
        }
    }

    private async Task AddToCartAsync()
    {
        if (!isLoggedIn)
        {
            Nav.NavigateTo(Pages.User.LogIn.Url(Nav));
            return;
        }

        if (item is not null)
        {
            // await BasketState.AddAsync(item);
            await UpdateNumInCartAsync();
        }
    }

    private async Task UpdateNumInCartAsync()
    {
        // var items = await BasketState.GetBasketItemsAsync();
        // numInCart = items.FirstOrDefault(row => row.ProductId == ItemId)?.Quantity ?? 0;
    }
}
```

The above code is a **Blazor component** written in **Razor syntax**

It is designed to **display the details of a catalog item (product)** and provide functionality to **add the item to a shopping cart** (not yet implemented in this sample)

Below is a brief explanation of the key components and functionality:

**Page Directive**

@page "/item/{itemId:int}": Defines the route for this page, accepting an integer parameter itemId from the URL

**Injected Dependencies**

@inject CatalogService CatalogService: Injects a service for fetching catalog items

@inject NavigationManager Nav: Provides navigation functionality

@inject IProductImageUrlProvider ProductImages: Provides URLs for product images

**UI Logic**

**When the item is found**:

Title and Header Section: Displays the item's name and brand in the page title and header sections

Item Details: Displays an image, description, brand, and price of the product

Includes a form for adding the item to the cart

If the user is logged in, it shows an "Add to shopping bag" button

If not logged in, it prompts the user to log in to make a purchase

Shopping Cart Info: If the item is already in the cart, it shows the quantity and a link to the cart

**When the item is not found**: Displays a "Not Found" message

**Code Section (@code)**

**Fields:**

item: Stores the catalog item details fetched from the service

numInCart: Stores the quantity of the item in the cart

isLoggedIn: Indicates if the user is logged in

notFound: Flags whether the item was not found

**Parameters and Cascading Parameters**:

ItemId: Captures the itemId from the URL

HttpContext: Provides access to the HTTP context, including user authentication status

**Lifecycle Method**:

OnInitializedAsync: Fetches the item details using the CatalogService. Updates the quantity in the cart

If the item is not found, sets a 404 Not Found status

**Event Handlers**:

AddToCartAsync: Adds the item to the cart. If the user is not logged in, navigates to the login page

UpdateNumInCartAsync: Updates the quantity of the item in the cart (currently commented out)

**Key Features**
Dynamic Content: Displays either product details or a "Not Found" message based on whether the item exists

User Authentication: Customizes the add-to-cart behavior based on the user's login status

Error Handling: Sets a 404 Not Found status code when the product is not available

Integration with Services: Uses CatalogService to fetch product details and a (commented-out) BasketState for cart operations

**IMPORTANT NOTE:**

Parts of the functionality, such as **updating the cart** quantity or **adding items to the cart**, are commented out, indicating they might be a work in progress or require additional implementation

## 2. We run the application and verify the results

When we run the application it first appears the Aspire Dashboard

![image](https://github.com/user-attachments/assets/212a50c2-3855-4016-ad4c-ec748d898618)

We navigate to the **WebApp** (https://localhost:7112/) and click on one item picture, we will be redirect to the **Item Page**

![image](https://github.com/user-attachments/assets/a11742fa-87ad-4b23-a8a2-aa3159e037e2)

Now we can visualize the **Item Page**

![image](https://github.com/user-attachments/assets/84e66bba-76f1-48dd-9cd6-6f58484975b5)

If we previously did **Not Logged** in the application we cannot see the **Add to Cart** button

Then we can go back to the **main Catalogue** web page and **Login** 

![image](https://github.com/user-attachments/assets/81c154e1-d866-4b6e-9f82-d50d3750b8a4)

After pressing on the **Login button** we are redirect to the **Login** web page

We enter the **Username** and **Password** and then press the **Login button**

![image](https://github.com/user-attachments/assets/da740d9b-a8fa-4f8b-888c-f116c8d9423e)

Now we are redirect to the **Catalogue** main web page and here we can press again in the bag picture to navigate to the **Item Page**

![image](https://github.com/user-attachments/assets/2bf99784-2db0-4203-8d96-d15026d54956)

## 3. A deeper explanation about the Catalogue main web page

The Catalogue web page is implemented in the **Catalog.razor** component

![image](https://github.com/user-attachments/assets/59725461-d58f-4f68-aa30-7e43a05b4d46)

Let's review the **Catalog.razor** component code:

```razor
@using Microsoft.AspNetCore.Components.Sections
@using WebAppComponents.Catalog
@page "/"

@inject NavigationManager Nav
@inject CatalogService CatalogService
@attribute [StreamRendering]

<PageTitle>AdventureWorks</PageTitle>
<SectionContent SectionName="page-header-title">Ready for a new adventure?</SectionContent>
<SectionContent SectionName="page-header-subtitle">Start the season with the latest in clothing and equipment.</SectionContent>

<div class="catalog">
    <CatalogSearch BrandId="@BrandId" ItemTypeId="@ItemTypeId" />

    @if (catalogResult is null)
    {
        <p>Loading...</p>
    }
    else
    {
        <div>
            <div class="catalog-items">
                @foreach (var item in catalogResult.Data)
                {
                    <CatalogListItem Item="@item" />
                }
            </div>

            <div class="page-links">
                @foreach (var pageIndex in GetVisiblePageIndexes(catalogResult))
                {
                    <NavLink ActiveClass="active-page" Match="@NavLinkMatch.All" href="@Nav.GetUriWithQueryParameter("page", pageIndex == 1 ? null : pageIndex)">@pageIndex</NavLink>
                }
            </div>
        </div>
    }
</div>

@code {
    const int PageSize = 9;

    [SupplyParameterFromQuery]
    public int? Page { get; set; }

    [SupplyParameterFromQuery(Name = "brand")]
    public int? BrandId { get; set; }

    [SupplyParameterFromQuery(Name = "type")]
    public int? ItemTypeId { get; set; }

    CatalogResult? catalogResult;

    static IEnumerable<int> GetVisiblePageIndexes(CatalogResult result)
        => Enumerable.Range(1, (int)Math.Ceiling(1.0 * result.Count / PageSize));

    protected override async Task OnInitializedAsync()
    {
        catalogResult = await CatalogService.GetCatalogItems(
            Page.GetValueOrDefault(1) - 1,
            PageSize,
            BrandId,
            ItemTypeId);
    }
}
```

This code defines a Blazor component that represents a **product catalog page** for a web application

This component leverages Blazor's ability to build interactive web UIs with rich data-binding and state management

It focuses on modularity and reusability, seen in the use of components like **CatalogSearch** and **CatalogListItem**

**Key Functionalities**

Displays a searchable, paginated product catalog

Dynamically updates based on URL query parameters (e.g., ?page=2&brand=1)

Efficiently fetches data using the CatalogService and renders it

Here's a brief explanation of its structure and functionality:

**Imports and Dependencies**

@using directives: Includes namespaces for additional functionality

Microsoft.AspNetCore.Components.Sections: Manages page sections

WebAppComponents.Catalog: Likely contains catalog-related components and services

@page directive: Declares the page's route as /

@inject directives: Injects dependencies for: NavigationManager (Nav), helps navigate programmatically and manipulate URLs and CatalogService (CatalogService), provides catalog data

**Attributes**

[StreamRendering]: Optimizes rendering for performance (used for streaming updates)

**Page Header Content**

<PageTitle>: Sets the browser tab title

<SectionContent>: Defines header sections (Title: "Ready for a new adventure?" and Subtitle: "Start the season with the latest in clothing and equipment.")

**Catalog Layout**

**Search Bar**:

<CatalogSearch>: Displays a search component for filtering by brand or item type.

**Catalog Loading and Display**: If catalogResult is null, a "Loading..." message is displayed

Otherwise:

Items Display: A grid or list of items rendered using <CatalogListItem> for each item in catalogResult.Data

Pagination: Links for pagination, allowing navigation between pages. The current page is highlighted using the ActiveClass="active-page" attribute.

**Code Section (@code)**

**Constants and Parameters**:

PageSize: Sets the number of items displayed per page (9)

Query parameters ([SupplyParameterFromQuery]): Maps query strings from the URL to Blazor component properties:

Page: Current page index

BrandId and ItemTypeId: Filter values

**Data Model**:

catalogResult: Holds the catalog data retrieved from the service

GetVisiblePageIndexes: Calculates the visible page numbers based on total items and PageSize

**Lifecycle Method**:

OnInitializedAsync: Fetches catalog data from the CatalogService asynchronously when the component initializes, based on the current page and filters

## 4. A deeper explanation about how to navigate from Catalogue web page to the Item Page






