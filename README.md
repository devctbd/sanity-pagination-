<!-- TABLE OF CONTENTS -->

<details>
  <summary >Snity Query</summary>
  <pre>
    
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

  </pre>
</details>



<details>
  <summary >Card Components</summary>
  <pre>
    
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

  </pre>
</details>
