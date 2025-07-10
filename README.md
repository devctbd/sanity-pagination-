<!-- TABLE OF CONTENTS -->

<details>
  <summary>Snity Query</summary>
  <pre>
    <code>
      export const getAllPostsQuery = (page: number, limit: number) =>
        defineQuery(
          `*[_type == "post" && defined(slug.current)]  |order(publishedAt desc)[${(page - 1) *
            limit}...${page * limit}]{
            _id,
            title,
            slug,
            mainImage,
            publishedAt,    
            body,
            short,
            readTime,
            author->{
                _id,
                name,
                image
            },
            "categories": categories[]->title,
            "paginationView": paginationView->postViewNumberPerPage
          }`
        );

    </code>

  </pre>
</details>

<details>
  <summary>Blog Page</summary>
  <pre>
    <code>
      import React, { PropsWithChildren } from "react";
      import { sanityFetch } from "@/sanity/lib/live";
      import { getAllPostsQuery } from "@/sanity/lib/queries";

      import {
        Breadcrumb,
        BreadcrumbItem,
        BreadcrumbLink,
        BreadcrumbList,
        BreadcrumbPage,
        BreadcrumbSeparator,
      } from "@/components/ui/breadcrumb";
      import ContactUsForSideBar from "@/components/ContactUsForSideBar";
      import PopulerPost from "@/components/PopulerPost";
      import GitHubUser from "@/components/GitHubUser";
      import PaginitionSection from "@/components/PaginitionSection";
      import HeaderSearchBar from "@/components/HeaderSearchBar";
      import CardComponents from "@/components/CardComponents";

      interface Props {
        searchParams: Promise<{ page?: string }>;
      }

      export default async function BlogPage({ searchParams }: Props) {
        // First get all posts to calculate total pages
        const { data: allPosts } = await sanityFetch({
          query: getAllPostsQuery(1, 1000), // Get all posts in one query
        });

        // Get the current page number from the search params, default to 1 if not provided
        const params = await searchParams;
        const currentPage = Number(params?.page) || 1;

        // Get the pagination view document to get the number of posts per page
        const { data: paginationView } = await sanityFetch({
          query: `*[_type == "paginationView"][0]`,
        });

        // Calculate the total number of pages based on the number of posts and posts per page
        const postsPerPage = paginationView?.postViewNumberPerPage || 2;
        const totalPages = Math.ceil(allPosts.length / postsPerPage);

        // Now get the current page's posts
        const { data: posts } = await sanityFetch({
          query: getAllPostsQuery(currentPage, postsPerPage),
        });

        // Create pagination data structure
        const pagination = {
          currentPage,
          totalPages,
          prev: currentPage > 1 ? currentPage - 1 : undefined,
          next: currentPage < totalPages ? currentPage + 1 : undefined,
          pages: Array.from({ length: totalPages }, (_, i) => i + 1),
        } as const;

        return (
          <div className="container mx-auto">
            <div
              className="mb-4 py-2 px-2 m-5 bg-white dark:bg-gray-800 rounded-lg shadow-md"
            >
              <Breadcrumb>
                <BreadcrumbList>
                  <BreadcrumbItem>
                    <BreadcrumbLink href="/">Home</BreadcrumbLink>
                  </BreadcrumbItem>
                  <BreadcrumbSeparator />
                  <BreadcrumbItem>
                    <BreadcrumbLink href="/blog">Blog</BreadcrumbLink>
                  </BreadcrumbItem>
                  <BreadcrumbSeparator />
                  <BreadcrumbItem>
                    <BreadcrumbPage>Blog</BreadcrumbPage>
                  </BreadcrumbItem>
                </BreadcrumbList>
              </Breadcrumb>
            </div>
            <div className="grid grid-cols-1 md:grid-cols-[1fr_300px] gap-4 lg:gap-12 p-2">
              <CardComponents posts={posts} />
              <div className="flex flex-col p-4">
                <div className="p-2 flex items-center justify-center">
                  <HeaderSearchBar />
                </div>

                <div className="p-2">
                  <ContactUsForSideBar />
                  <PopulerPost />
                </div>
              </div>
            </div>
            <div className="my-5">
              <PaginitionSection pagination={pagination} />
            </div>
            <div className="my-5">
              <GitHubUser />
            </div>
          </div>
        );
      }

    </code>

  </pre>
</details>

<details>
  <summary>Card Components</summary>
  <pre>
    <code>
      import React from "react";
      import {
        Pagination,
        PaginationContent,
        PaginationItem,
        PaginationLink,
        PaginationNext,
        PaginationPrevious,
        PaginationEllipsis,
      } from "@/components/ui/pagination";

      interface PaginationProps {
        pagination: {
          currentPage: number;
          totalPages: number;
          prev: number | undefined;
          next: number | undefined;
          pages: number[];
        };
      }

      /**
       * This component renders a pagination section. It takes a pagination object as a prop which
       * contains the current page, the total number of pages, the previous page, and the next page.
       * It renders the pagination links and the ellipsis.
       *
       * @param pagination - The pagination object.
       * @returns A React component.
       */
      const PaginitionSection = ({ pagination }: PaginationProps) => {
        return (
          <div className="my-5">
            <Pagination>
              <PaginationContent>
                <PaginationItem>
                  {/**
                   * If there is a previous page, render a link to it.
                   */}
                  <PaginationPrevious
                    href={pagination.prev ? `?page=${pagination.prev}` : undefined}
                  />
                </PaginationItem>
                {/**
                 * Show first page
                 */}
                <PaginationItem>
                  {/**
                   * If the current page is not the first page, render a link to the first page.
                   */}
                  <PaginationLink
                    href={pagination.currentPage === 1 ? undefined : "?page=1"}
                    className={
                      pagination.currentPage === 1
                        ? "bg-gray-700 text-white"
                        : "text-gray-700"
                    }
                  >
                    1
                  </PaginationLink>
                </PaginationItem>

                {pagination.totalPages > 2 && (
                  <PaginationItem>
                    {/**
                     * If there are more than 2 pages, render an ellipsis.
                     */}
                    <PaginationEllipsis />
                  </PaginationItem>
                )}

                <PaginationItem>
                  {/**
                   * If the current page is not the last page, render a link to the last page.
                   */}
                  <PaginationLink
                    href={
                      pagination.currentPage === pagination.totalPages
                        ? undefined
                        : `?page=${pagination.totalPages}`
                    }
                    className={
                      pagination.currentPage === pagination.totalPages
                        ? "bg-gray-700 text-white"
                        : "text-gray-700"
                    }
                  >
                    {pagination.totalPages}
                  </PaginationLink>
                </PaginationItem>
                <PaginationItem>
                  {/**
                   * If there is a next page, render a link to it.
                   */}
                  <PaginationNext
                    href={pagination.next ? `?page=${pagination.next}` : undefined}
                  />
                </PaginationItem>
              </PaginationContent>
            </Pagination>
          </div>
        );
      };

      export default PaginitionSection;

    </code>

  </pre>
</details>

