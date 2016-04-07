---
layout: post
title: "grape + grape_entity api"
description: ""
category:
tags: [grape rails grape_entity api]
---
{% include JB/setup %}

```ruby
class V1::WishLists < Grape::API
  helpers V1::Modules::SharedParams

  resource :wish_lists do

    desc "Get current user wish lists"
    get do
      authenticate!

      wish_lists = current_user.wish_lists.recent # FIXME: n+1
      present wish_lists, with: V1::Entities::WishList, expose_product: true
    end

    desc "Create Wish List"
    params do
      requires :title, type: String, desc: "Wish list title"
      optional :anniversary_on, type: String, desc: "Anniversary format (yyyy-mm-dd)"
    end
    post do
      authenticate!

      wish_list_params = declared(params, include_missing: false)
      wish_list = current_user.wish_lists.build(wish_list_params)
      wish_list.save!

      present wish_list, with: V1::Entities::WishList
    end

    route_param :id do
      desc "Get wish list products"
      params do
        use :pagination
      end
      get "wish_list_products" do
        authenticate!
        wish_list = WishList.find(params[:id])
        authorize!(wish_list, :get_variants?)

        products = wish_list.wish_list_products
          .recent
          .page(params[:page])
          .per(params[:per_page])

        present products, with: V1::Entities::WishListProduct
      end

      desc "Update Wish List"
      params do
        requires :title, type: String, desc: "Wish list title"
        optional :anniversary_on, type: String, desc: "Anniversary format (yyyy-mm-dd)"
      end
      patch do
        authenticate!
        wish_list = current_user.wish_lists.find(params[:id])
        authorize!(wish_list, :update?)

        wish_list_params = declared(params, include_missing: false)

        wish_list.update!(wish_list_params)

        present wish_list, with: V1::Entities::WishList
      end

      desc "Delete Wish List"
      delete do
        authenticate!
        wish_list = current_user.wish_lists.find(params[:id])
        authorize!(wish_list, :destroy?)

        wish_list.destroy
        body false
      end

      desc "Get Wish List"
      get do
        authenticate!
        wish_list = WishList.find(params[:id])
        authorize!(wish_list, :show?)

        present wish_list, with: V1::Entities::WishList
      end

    end



  end
end
```
