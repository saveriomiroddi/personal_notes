# Algorithms

The implementations are not intended to be fully optimized.

## All permutations of a sequence

Swap the leftmost element of each subarray with each other element, and for each swap, recurse on the subarray starting at the next position.

Complexity: O(n!), however, if the block processes each permuted array by iterating it, it becomes O(n * n!); in this case, it's possible to reduce the complexity back to O(n!) by integrating the processing in this routine, at the cost of readability/coupling.

```rb
# Execute the block on each permutation.
def permutations(sequence, start_position = 0, &block)
  if start_position == sequence.size
    yield sequence
    return # technically optional, but confusing if omitted
  end

  start_position.upto(sequence.size - 1) do |new_position|
    sequence[start_position], sequence[new_position] = sequence[new_position], sequence[start_position]

    permutations(sequence, start_position + 1, &block)

    # By restoring the sequence, we don't need to use a temporary array.
    sequence[start_position], sequence[new_position] = sequence[new_position], sequence[start_position]
  end
end
```