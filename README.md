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


